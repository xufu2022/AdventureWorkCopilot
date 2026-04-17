using Xunit;
using FluentAssertions;
using Microsoft.Extensions.Configuration;
using Microsoft.Data.SqlClient;
using System.Data;
using System.IO;

namespace TestProject.StoredProcedures
{
    public class [SP_NAME]_Tests : IDisposable
    {
        private readonly string _connectionString;
        private SqlConnection _connection;
        private SqlTransaction _transaction;

        public [SP_NAME]_Tests()
        {
            var config = new ConfigurationBuilder()
                .SetBasePath(Directory.GetCurrentDirectory())
                .AddJsonFile("appsettings.json")
                .Build();
            
            _connectionString = config.GetConnectionString("DefaultConnection");
            _connection = new SqlConnection(_connectionString);
            _connection.Open();
            _transaction = _connection.BeginTransaction();
        }

        [Fact]
        public void TestCase1_Insert_AddsRecordSuccessfully()
        {
            using var cmd = new SqlCommand("[SP_NAME]", _connection, _transaction);
            cmd.CommandType = CommandType.StoredProcedure;
            cmd.Parameters.AddWithValue("@Name", "Test User");
            cmd.Parameters.AddWithValue("@Email", "test@example.com");
            
            var outputParam = new SqlParameter("@Id", SqlDbType.Int)
            {
                Direction = ParameterDirection.Output
            };
            cmd.Parameters.Add(outputParam);
            
            cmd.ExecuteNonQuery();
            var newId = (int)outputParam.Value;
            
            using var verifyCmd = new SqlCommand("SELECT COUNT(*) FROM Users WHERE Id = @Id", _connection, _transaction);
            verifyCmd.Parameters.AddWithValue("@Id", newId);
            var count = (int)verifyCmd.ExecuteScalar();
            count.Should().Be(1);
        }

        [Fact]
        public void TestCase2_Update_ModifiesExistingRecord()
        {
            int testId = InsertTestRecord();
            
            using var cmd = new SqlCommand("[SP_NAME]", _connection, _transaction);
            cmd.CommandType = CommandType.StoredProcedure;
            cmd.Parameters.AddWithValue("@Id", testId);
            cmd.Parameters.AddWithValue("@Name", "Updated Name");
            cmd.Parameters.AddWithValue("@Email", "updated@example.com");
            cmd.ExecuteNonQuery();
            
            using var verifyCmd = new SqlCommand("SELECT Name, Email FROM Users WHERE Id = @Id", _connection, _transaction);
            verifyCmd.Parameters.AddWithValue("@Id", testId);
            using var reader = verifyCmd.ExecuteReader();
            reader.Read();
            reader["Name"].ToString().Should().Be("Updated Name");
            reader["Email"].ToString().Should().Be("updated@example.com");
        }

        [Fact]
        public void TestCase3_Delete_RemovesRecordSuccessfully()
        {
            int testId = InsertTestRecord();
            
            using var cmd = new SqlCommand("[SP_NAME]", _connection, _transaction);
            cmd.CommandType = CommandType.StoredProcedure;
            cmd.Parameters.AddWithValue("@Id", testId);
            cmd.ExecuteNonQuery();
            
            using var verifyCmd = new SqlCommand("SELECT COUNT(*) FROM Users WHERE Id = @Id", _connection, _transaction);
            verifyCmd.Parameters.AddWithValue("@Id", testId);
            var count = (int)verifyCmd.ExecuteScalar();
            count.Should().Be(0);
        }

        private int InsertTestRecord()
        {
            using var cmd = new SqlCommand(
                "INSERT INTO Users (Name, Email) VALUES ('Temp User', 'temp@test.com'); SELECT SCOPE_IDENTITY();", 
                _connection, 
                _transaction);
            return Convert.ToInt32(cmd.ExecuteScalar());
        }

        public void Dispose()
        {
            _transaction?.Rollback();
            _connection?.Close();
            _connection?.Dispose();
        }
    }
}