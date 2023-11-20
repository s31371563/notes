import com.amazonaws.services.dynamodbv2.datamodeling.DynamoDBMapper;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import java.io.IOException;
import java.nio.file.Files;
import java.nio.file.Paths;
import java.util.List;
import java.util.Map;

@Service
public class DynamoDBService {

    @Autowired
    private DynamoDBMapper dynamoDBMapper;

    public void performOperationsFromFile(String filePath) throws IOException {
        String jsonData = new String(Files.readAllBytes(Paths.get(filePath)));
        List<Map<String, Object>> operations = parseJsonOperations(jsonData);

        for (Map<String, Object> operation : operations) {
            String operationType = (String) operation.get("operation");
            String tableName = (String) operation.get("tableName");

            switch (operationType) {
                case "insert":
                    List<Map<String, Object>> insertItems = (List<Map<String, Object>>) operation.get("items");
                    batchSave(tableName, insertItems);
                    break;
                case "update":
                    List<Map<String, Object>> updateItems = (List<Map<String, Object>>) operation.get("updates");
                    for (Map<String, Object> updateItem : updateItems) {
                        dynamoDBMapper.save(updateItem);
                    }
                    break;
                case "delete":
                    List<Map<String, Object>> deleteKeys = (List<Map<String, Object>>) operation.get("keys");
                    for (Map<String, Object> deleteKey : deleteKeys) {
                        dynamoDBMapper.delete(deleteKey);
                    }
                    break;
                default:
                    System.out.println("Unsupported operation type: " + operationType);
            }
        }
    }

    private List<Map<String, Object>> parseJsonOperations(String jsonData) throws IOException {
        ObjectMapper objectMapper = new ObjectMapper();
        return objectMapper.readValue(jsonData, List.class);
    }

    private void batchSave(String tableName, List<Map<String, Object>> items) {
        List<Object> entities = items.stream()
                .map(item -> dynamoDBMapper.map(item, YourEntityClass.class))
                .collect(Collectors.toList());

        dynamoDBMapper.batchSave(entities);
    }
}


[
    {
        "operation": "insert",
        "tableName": "YourTableName",
        "items": [
            {
                "id": "1",
                "name": "John"
            },
            {
                "id": "2",
                "name": "Jane"
            }
        ]
    },
    {
        "operation": "update",
        "tableName": "YourTableName",
        "updates": [
            {
                "key": {"id": "1"},
                "updateExpression": {
                    "SET": {
                        "name": "UpdatedName"
                    }
                }
            },
            // Add more update items as needed
        ]
    },
    {
        "operation": "delete",
        "tableName": "YourTableName",
        "keys": [
            {"id": "1"},
            {"id": "2"}
            // Add more keys for deletion as needed
        ]
    },
    // Add more operations as needed
]
	
	
	
