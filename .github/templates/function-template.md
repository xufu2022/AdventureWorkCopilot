using Xunit;
using FluentAssertions;
using Microsoft.Extensions.Configuration;
using Microsoft.Data.SqlClient;
using System.Data;
using System.IO;

namespace TestProject.Functions
{
    public class [FUNCTION_NAME]_Tests : IDisposable
    {
        private readonly string _connectionString;
        private SqlConnection _connection;

        public [FUNCTION_NAME]_Tests()
        {
            var config = new ConfigurationBuilder()
                .SetBasePath(Directory.GetCurrentDirectory())
                .AddJsonFile("appsettings.json")
                .Build();
            
            _connectionString = config.GetConnectionString("DefaultConnection");
            _connection = new SqlConnection(_connectionString);
            _connection.Open();
        }

        [Fact]
        public void TestCase1_ValidInput_ReturnsExpectedValue()
        {
            using var cmd = new SqlCommand("SELECT dbo.[FUNCTION_NAME](@Param1, @Param2)", _connection);
            cmd.Parameters.AddWithValue("@Param1", 100);
            cmd.Parameters.AddWithValue("@Param2", 50);
            
            var result = cmd.ExecuteScalar();
            
            result.Should().NotBeNull();
            result.Should().BeOfType<decimal>();
        }

        [Fact]
        public void TestCase2_NullInput_ReturnsNull()
        {
            using var cmd = new SqlCommand("SELECT dbo.[FUNCTION_NAME](@Param1, @Param2)", _connection);
            cmd.Parameters.AddWithValue("@Param1", DBNull.Value);
            cmd.Parameters.AddWithValue("@Param2", 50);
            
            var result = cmd.ExecuteScalar();
            
            result.Should().Be(DBNull.Value);
        }

        [Fact]
        public void TestCase3_EdgeCase_HandlesBoundaryValues()
        {
            using var cmd = new SqlCommand("SELECT dbo.[FUNCTION_NAME](@Param1, @Param2)", _connection);
            cmd.Parameters.AddWithValue("@Param1", 0);
            cmd.Parameters.AddWithValue("@Param2", 0);
            
            var result = cmd.ExecuteScalar();
            
            result.Should().NotBeNull();
        }

        [Fact]
        public void TestCase4_NegativeValues_HandlesCorrectly()
        {
            using var cmd = new SqlCommand("SELECT dbo.[FUNCTION_NAME](@Param1, @Param2)", _connection);
            cmd.Parameters.AddWithValue("@Param1", -50);
            cmd.Parameters.AddWithValue("@Param2", 25);
            
            var result = cmd.ExecuteScalar();
            
            result.Should().NotBeNull();
        }

        public void Dispose()
        {
            _connection?.Close();
            _connection?.Dispose();
        }
    }
}