using System;
using System.Data.SqlClient;
using System.Text;

public class CDCRollback
{
    private string connectionString = "Data Source=your_server;Initial Catalog=your_database;Integrated Security=True";

    public void RollbackChanges(string updateTs, int updatedPersonKey)
    {
        using (SqlConnection connection = new SqlConnection(connectionString))
        {
            connection.Open();

            // Step 1: Identify CDC Tables
            string[] cdcTables = GetCDCEnabledTables(connection);

            // Step 2: Retrieve CDC Data and Step 3: Generate Rollback Script
            StringBuilder rollbackScriptBuilder = new StringBuilder();
            foreach (string cdcTable in cdcTables)
            {
                string rollbackScript = GetRollbackScriptForTable(connection, cdcTable, updateTs, updatedPersonKey);
                rollbackScriptBuilder.AppendLine(rollbackScript);
            }

            // Step 4: Execute the Rollback Script
            SqlTransaction transaction = connection.BeginTransaction();
            try
            {
                SqlCommand cmd = connection.CreateCommand();
                cmd.Transaction = transaction;
                cmd.CommandText = rollbackScriptBuilder.ToString();
                cmd.ExecuteNonQuery();
                transaction.Commit();
                Console.WriteLine("Rollback successful.");
            }
            catch (Exception ex)
            {
                transaction.Rollback();
                Console.WriteLine("Rollback failed: " + ex.Message);
            }
        }
    }

    private string[] GetCDCEnabledTables(SqlConnection connection)
    {
        // Query the system tables to identify CDC-enabled tables.
        List<string> cdcTables = new List<string>();
        using (SqlCommand cmd = new SqlCommand("SELECT TABLE_NAME FROM INFORMATION_SCHEMA.COLUMNS WHERE COLUMN_NAME = 'updateTs' AND TABLE_NAME IN (SELECT TABLE_NAME FROM INFORMATION_SCHEMA.COLUMNS WHERE COLUMN_NAME = 'updatedPersonKey');", connection))
        {
            using (SqlDataReader reader = cmd.ExecuteReader())
            {
                while (reader.Read())
                {
                    cdcTables.Add(reader["TABLE_NAME"].ToString());
                }
            }
        }
        return cdcTables.ToArray();
    }

    private string GetRollbackScriptForTable(SqlConnection connection, string cdcTable, string updateTs, int updatedPersonKey)
    {
        StringBuilder rollbackScriptBuilder = new StringBuilder();

        // Prepare the CDC function name based on the CDC table name
        string cdcFunctionName = "cdc.fn_cdc_get_all_changes_" + cdcTable;

        // Query the CDC data for the given table and filter by updateTs and updatedPersonKey
        string queryString = $"SELECT __$operation, * FROM {cdcFunctionName}(@from_lsn, @to_lsn, N'all update old') " +
                             "WHERE updateTs = @updateTs AND updatedPersonKey = @updatedPersonKey " +
                             "ORDER BY __$seqval DESC;"; // <-- Retrieve data in descending order of CDC timestamp

        using (SqlCommand cmd = new SqlCommand(queryString, connection))
        {
            cmd.Parameters.AddWithValue("@from_lsn", SqlDbType.Binary).Value = DBNull.Value;
            cmd.Parameters.AddWithValue("@to_lsn", SqlDbType.Binary).Value = DBNull.Value;
            cmd.Parameters.AddWithValue("@updateTs", SqlDbType.DateTime).Value = DateTime.Parse(updateTs);
            cmd.Parameters.AddWithValue("@updatedPersonKey", SqlDbType.Int).Value = updatedPersonKey;

            using (SqlDataReader reader = cmd.ExecuteReader())
            {
                while (reader.Read())
                {
                    string operation = reader["__$operation"].ToString();

                    switch (operation)
                    {
                        case "1": // Delete
                            rollbackScriptBuilder.AppendLine($"INSERT INTO {cdcTable} SELECT * FROM cdc.{cdcTable}_CT WHERE __$seqval = {reader["__$seqval"]};");
                            break;
                        case "2": // Insert
                            rollbackScriptBuilder.AppendLine($"DELETE FROM {cdcTable} WHERE __$seqval = {reader["__$seqval"]};");
                            break;
                        case "3": // Update
                            rollbackScriptBuilder.AppendLine($"UPDATE {cdcTable} SET {GetColumnUpdatesForUpdate(reader)} WHERE __$seqval = {reader["__$seqval"]};");
                            break;
                        default:
                            break;
                    }
                }
            }
        }

        return rollbackScriptBuilder.ToString();
    }

    private string GetColumnUpdatesForUpdate(SqlDataReader reader)
    {
        // Assuming you have columns other than __$seqval, updateTs, and updatedPersonKey.
        // You need to exclude these columns from the update statement.
        StringBuilder columnUpdatesBuilder = new StringBuilder();
        for (int i = 0; i < reader.FieldCount; i++)
        {
            string columnName = reader.GetName(i);
            if (columnName != "__$seqval" && columnName != "updateTs" && columnName != "updatedPersonKey" && reader[i] != DBNull.Value)
            {
                columnUpdatesBuilder.Append($"{columnName} = '{reader[i]}', ");
            }
        }

        // Remove the trailing comma and space
        if (columnUpdatesBuilder.Length > 0)
        {
            columnUpdatesBuilder.Length -= 2;
        }

        return columnUpdatesBuilder.ToString();
    }
}

    private List<string> GetPrimaryKeyColumns(SqlConnection connection, string tableName)
    {
        // Query the system tables to get the primary key columns for the given table
        List<string> primaryKeyColumns = new List<string>();
        using (SqlCommand cmd = new SqlCommand($"SELECT COLUMN_NAME FROM INFORMATION_SCHEMA.KEY_COLUMN_USAGE WHERE OBJECTPROPERTY(OBJECT_ID(CONSTRAINT_SCHEMA + '.' + CONSTRAINT_NAME), 'IsPrimaryKey') = 1 AND TABLE_NAME = '{tableName}';", connection))
        {
            using (SqlDataReader reader = cmd.ExecuteReader())
            {
                while (reader.Read())
                {
                    primaryKeyColumns.Add(reader["COLUMN_NAME"].ToString());
                }
            }
        }
        return primaryKeyColumns;
    }



using System;
using System.Collections.Generic;
using System.Data;
using System.Data.SqlClient;
using System.Text;

public class CDCRollback
{
    private string connectionString = "Data Source=your_server;Initial Catalog=your_database;Integrated Security=True";

    public void RollbackChanges(string updateTs, int updatedPersonKey)
    {
        using (SqlConnection connection = new SqlConnection(connectionString))
        {
            connection.Open();

            // Step 1: Identify CDC Tables
            string[] cdcTables = GetCDCEnabledTables(connection);

            // Step 2: Retrieve CDC Data and Step 3: Generate Rollback Script
            StringBuilder rollbackScriptBuilder = new StringBuilder();
            foreach (string cdcTable in cdcTables)
            {
                string rollbackScript = GetRollbackScriptForTable(connection, cdcTable, updateTs, updatedPersonKey);
                rollbackScriptBuilder.AppendLine(rollbackScript);
            }

            // Step 4: Execute the Rollback Script
            SqlTransaction transaction = connection.BeginTransaction();
            try
            {
                SqlCommand cmd = connection.CreateCommand();
                cmd.Transaction = transaction;
                cmd.CommandText = rollbackScriptBuilder.ToString();
                cmd.ExecuteNonQuery();
                transaction.Commit();
                Console.WriteLine("Rollback successful.");
            }
            catch (Exception ex)
            {
                transaction.Rollback();
                Console.WriteLine("Rollback failed: " + ex.Message);
            }
        }
    }

    private string[] GetCDCEnabledTables(SqlConnection connection)
    {
        // Query the system tables to identify CDC-enabled tables.
        List<string> cdcTables = new List<string>();
        using (SqlCommand cmd = new SqlCommand("SELECT TABLE_NAME FROM INFORMATION_SCHEMA.COLUMNS WHERE COLUMN_NAME = 'updateTs' AND TABLE_NAME IN (SELECT TABLE_NAME FROM INFORMATION_SCHEMA.COLUMNS WHERE COLUMN_NAME = 'updatedPersonKey');", connection))
        {
            using (SqlDataReader reader = cmd.ExecuteReader())
            {
                while (reader.Read())
                {
                    cdcTables.Add(reader["TABLE_NAME"].ToString());
                }
            }
        }
        return cdcTables.ToArray();
    }

    private string GetRollbackScriptForTable(SqlConnection connection, string cdcTable, string updateTs, int updatedPersonKey)
    {
        StringBuilder rollbackScriptBuilder = new StringBuilder();

        // Prepare the CDC function name based on the CDC table name
        string cdcFunctionName = "cdc.fn_cdc_get_all_changes_" + cdcTable;

        // Identify the primary key columns for the table
        List<string> primaryKeyColumns = GetPrimaryKeyColumns(connection, cdcTable);

        // Query the CDC data for the given table and filter by updateTs and updatedPersonKey
        string queryString = $"SELECT __$operation, * FROM {cdcFunctionName}(@from_lsn, @to_lsn, N'all update old') " +
                             "WHERE updateTs = @updateTs AND updatedPersonKey = @updatedPersonKey " +
                             "ORDER BY __$seqval DESC;";

        using (SqlCommand cmd = new SqlCommand(queryString, connection))
        {
            cmd.Parameters.AddWithValue("@from_lsn", SqlDbType.Binary).Value = DBNull.Value;
            cmd.Parameters.AddWithValue("@to_lsn", SqlDbType.Binary).Value = DBNull.Value;
            cmd.Parameters.AddWithValue("@updateTs", SqlDbType.DateTime).Value = DateTime.Parse(updateTs);
            cmd.Parameters.AddWithValue("@updatedPersonKey", SqlDbType.Int).Value = updatedPersonKey;

            using (SqlDataReader reader = cmd.ExecuteReader())
            {
                while (reader.Read())
                {
                    string operation = reader["__$operation"].ToString();

                    switch (operation)
                    {
                        case "1": // Delete
                            rollbackScriptBuilder.AppendLine($"INSERT INTO {cdcTable} SELECT * FROM cdc.{cdcTable}_CT WHERE __$seqval = {reader["__$seqval"]};");
                            break;
                        case "2": // Insert
                            rollbackScriptBuilder.AppendLine($"DELETE FROM {cdcTable} WHERE __$seqval = {reader["__$seqval"]};");
                            break;
                        case "3": // Update
                            rollbackScriptBuilder.AppendLine($"UPDATE {cdcTable} SET {GetColumnUpdatesForUpdate(reader, primaryKeyColumns)} WHERE __$seqval = {reader["__$seqval"]};");
                            break;
                        default:
                            break;
                    }
                }
            }
        }

        return rollbackScriptBuilder.ToString();
    }

    private List<string> GetPrimaryKeyColumns(SqlConnection connection, string tableName)
    {
        // Query the system tables to get the primary key columns for the given table
        List<string> primaryKeyColumns = new List<string>();
        using (SqlCommand cmd = new SqlCommand($"SELECT COLUMN_NAME FROM INFORMATION_SCHEMA.KEY_COLUMN_USAGE WHERE OBJECTPROPERTY(OBJECT_ID(CONSTRAINT_SCHEMA + '.' + CONSTRAINT_NAME), 'IsPrimaryKey') = 1 AND TABLE_NAME = '{tableName}';", connection))
        {
            using (SqlDataReader reader = cmd.ExecuteReader())
            {
                while (reader.Read())
                {
                    primaryKeyColumns.Add(reader["COLUMN_NAME"].ToString());
                }
            }
        }
        return primaryKeyColumns;
    }

    private string GetColumnUpdatesForUpdate(SqlDataReader reader, List<string> primaryKeyColumns)
    {
        // Exclude the primary key columns from the update statement.
        StringBuilder columnUpdatesBuilder = new StringBuilder();
        for (int i = 0; i < reader.FieldCount; i++)
        {
            string columnName = reader.GetName(i);
            if (!primaryKeyColumns.Contains(columnName) && reader[i] != DBNull.Value)
            {
                columnUpdatesBuilder.Append($"{columnName} = '{reader[i]}', ");
            }
        }

        // Remove the trailing comma and space
        if (columnUpdatesBuilder.Length > 0)
        {
            columnUpdatesBuilder.Length -= 2;
        }

        return columnUpdatesBuilder.ToString();
    }
}

    private string GetPrimaryKeyConditionsForUpdate(SqlDataReader reader, List<string> primaryKeyColumns)
    {
        // Create the conditions for the primary key columns to be used in UPDATE statement
        StringBuilder primaryKeyConditionsBuilder = new StringBuilder();
        foreach (var column in primaryKeyColumns)
        {
            primaryKeyConditionsBuilder.Append($"{column} = '{reader[column]}' AND ");
        }

        // Remove the trailing "AND"
        if (primaryKeyConditionsBuilder.Length > 0)
        {
            primaryKeyConditionsBuilder.Length -= 5;
        }

        return primaryKeyConditionsBuilder.ToString();
    }

  case "1": // Delete
                            rollbackScriptBuilder.AppendLine($"INSERT INTO {cdcTable} ({string.Join(", ", primaryKeyColumns)}) SELECT {string.Join(", ", primaryKeyColumns)} FROM cdc.{cdcTable}_CT WHERE __$seqval = {reader["__$seqval"]};");
                            break;
                        case "2": // Insert
                            rollbackScriptBuilder.AppendLine($"DELETE FROM {cdcTable} WHERE {GetPrimaryKeyConditionsForDelete(reader, primaryKeyColumns)};");
                            break;
                        case "3": // Update
                            rollbackScriptBuilder.AppendLine($"UPDATE {cdcTable} SET {GetColumnUpdatesForUpdate(reader, primaryKeyColumns)} WHERE {GetPrimaryKeyConditionsForUpdate(reader, primaryKeyColumns)};");
                            break;




  byte[] seqvalBytes = (byte[])reader["__$seqval"];

                        // Convert byte array to a long integer (int64)
                        long seqval = BitConverter.ToInt64(seqvalBytes, 0);
