참고! 완성된 솔루션을 사용하려면 TestCLI의 `settings.json`에 있는 키로 다음을 입력해야 합니다.

```json
{
  "CognitiveServicesKeys": {
    "Vision": "VisionKeyHere"
  },
  "AzureStorage": {
    "ConnectionString": "ConnectionStringHere",
    "BlobContainer": "images"
  },
  "CosmosDB": {
    "EndpointURI": "CosmosURIHere",
    "Key": "CosmosKeyHere",
    "DatabaseName": "images",
    "CollectionName": "metadata"
  }
}
```