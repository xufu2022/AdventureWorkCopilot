using Xunit;
using FluentAssertions;
using Microsoft.Extensions.Configuration;
using Microsoft.Data.SqlClient;
using System.Data;
using System.Text.Json;
using System.IO;
using System.Collections.Generic;

namespace TestProject.StoredProcedures
{
    public class [SP_NAME]_Tests : IDisposable
    {
        private readonly string _connectionString;
        private SqlConnection _connection;
        private readonly List<TestCaseData> _testCases;

        public class TestCaseData
        {
            public int Id { get; set; }
            public string Name { get; set; }
            public string Description { get; set; }
            public Dictionary<string, object> Parameters { get; set; }
            public int? ExpectedRowCount { get; set; }
            public string ExpectedError { get; set; }
        }

        public [SP_NAME]_Tests()
        {
            var config = new ConfigurationBuilder()
                .SetBasePath(Directory.GetCurrentDirectory())
                .AddJsonFile("appsettings.json")
                .Build();
            
            _connectionString = config.GetConnectionString("DefaultConnection");
            _connection = new SqlConnection(_connectionString);
            _connection.Open();
            
            var jsonPath = Path.Combine("TestCases", "[SP_NAME]_Tests.json");
            var jsonContent = File.ReadAllText(jsonPath);
            var testData = JsonSerializer.Deserialize<Dictionary<string, List<TestCaseData>>>(jsonContent);
            _testCases = testData["testCases"];
        }

        [Fact]
        public void TestCase1_ValidInput_ReturnsData()
        {
            var testCase = _testCases[0];
            
            using var cmd = new SqlCommand("[SP_NAME]", _connection);
            cmd.CommandType = CommandType.StoredProcedure;
            foreach (var param in testCase.Parameters)
            {
                cmd.Parameters.AddWithValue(param.Key, param.Value ?? DBNull.Value);
            }
            
            using var reader = cmd.ExecuteReader();
            var results = new List<Dictionary<string, object>>();
            while (reader.Read())
            {
                var row = new Dictionary<string, object>();
                for (int i = 0; i < reader.FieldCount; i++)
                {
                    row[reader.GetName(i)] = reader.GetValue(i);
                }
                results.Add(row);
            }
            
            if (testCase.ExpectedRowCount.HasValue)
            {
                results.Count.Should().Be(testCase.ExpectedRowCount.Value);
            }
        }

        [Fact]
        public void TestCase2_InvalidInput_ReturnsEmpty()
        {
            var testCase = _testCases[1];
            
            using var cmd = new SqlCommand("[SP_NAME]", _connection);
            cmd.CommandType = CommandType.StoredProcedure;
            foreach (var param in testCase.Parameters)
            {
                cmd.Parameters.AddWithValue(param.Key, param.Value ?? DBNull.Value);
            }
            
            using var reader = cmd.ExecuteReader();
            var rowCount = 0;
            while (reader.Read()) rowCount++;
            
            rowCount.Should().Be(0);
        }

        [Fact]
        public void TestCase3_NullParameter_HandlesGracefully()
        {
            var testCase = _testCases[2];
            
            using var cmd = new SqlCommand("[SP_NAME]", _connection);
            cmd.CommandType = CommandType.StoredProcedure;
            foreach (var param in testCase.Parameters)
            {
                cmd.Parameters.AddWithValue(param.Key, param.Value ?? DBNull.Value);
            }
            
            var exception = Record.Exception(() => cmd.ExecuteReader());
            exception.Should().BeNull();
        }

        public void Dispose()
        {
            _connection?.Close();
            _connection?.Dispose();
        }
    }
}