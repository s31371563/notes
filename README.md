    public static void main(String[] args) throws IOException {
        String jsonFilePath = "path/to/operations.json";
        String jsonData = new String(Files.readAllBytes(Paths.get(jsonFilePath)));

        List<Map<String, Object>> operations = // Parse jsonData to a List<Map>

        for (Map<String, Object> operationData : operations) {
            performOperation(operationData);
        }
    }

    private static void performOperation(Map<String, Object> operationData) {
        String operationType = (String) operationData.get("operation");
        String tableName = (String) operationData.get("tableName");

        switch (operationType) {
            case "insert":
                performInsert(tableName, (List<Map<String, AttributeValue>>) operationData.get("items"));
                break;
            case "update":
                performUpdate(tableName, (Map<String, AttributeValue>) operationData.get("key"),
                        (Map<String, AttributeValueUpdate>) operationData.get("updateExpression"));
                break;
            case "delete":
                performDelete(tableName, (Map<String, AttributeValue>) operationData.get("key"));
                break;
            default:
                System.out.println("Unsupported operation type: " + operationType);
        }
    }

    private static void performInsert(String tableName, List<Map<String, AttributeValue>> items) {
        BatchWriteItemRequest batchWriteItemRequest = BatchWriteItemRequest.builder()
                .requestItems(Map.of(tableName, items))
                .build();

        dynamoDbClient.batchWriteItem(batchWriteItemRequest);
    }

    private static void performUpdate(String tableName, Map<String, AttributeValue> key,
                                      Map<String, AttributeValueUpdate> updateExpression) {
        UpdateItemRequest updateItemRequest = UpdateItemRequest.builder()
                .tableName(tableName)
                .key(key)
                .attributeUpdates(updateExpression)
                .build();

        dynamoDbClient.updateItem(updateItemRequest);
    }

    private static void performDelete(String tableName, Map<String, AttributeValue> key) {
        DeleteItemRequest deleteItemRequest = DeleteItemRequest.builder()
                .tableName(tableName)
                .key(key)
                .build();

        dynamoDbClient.deleteItem(deleteItemRequest);
    }
