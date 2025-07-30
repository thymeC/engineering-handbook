# Mongo Mock

## unittest
```python
from unittest import TestCase
from unittest.mock import Mock, patch
import mongomock
from bson import ObjectId
from datetime import datetime

# Assuming your mongo operations are in this file
from database.mongo_operations import MongoOperations

class TestMongoOperations(TestCase):
    def setUp(self):
        # Option 1: Using mongomock
        self.mock_client = mongomock.MongoClient()
        self.mock_db = self.mock_client['test_db']
        self.mongo_ops = MongoOperations(self.mock_db)

        # Insert some test data
        self.test_data = {
            "action_id": 1,
            "action_name": "test_action",
            "created_time": datetime.utcnow(),
            "status": "active"
        }
        self.collection = self.mock_db['actions']
        
    def test_create_document(self):
        # Test creating a single document
        result = self.mongo_ops.create_document(
            collection_name='actions',
            document=self.test_data
        )
        
        # Assert document was created
        self.assertIsNotNone(result)
        
        # Verify document exists in collection
        saved_doc = self.collection.find_one({"action_id": 1})
        self.assertEqual(saved_doc["action_name"], "test_action")

    def test_create_many_documents(self):
        documents = [
            {"action_id": i, "action_name": f"test_{i}"} 
            for i in range(3)
        ]
        
        result = self.mongo_ops.create_many_documents(
            collection_name='actions',
            documents=documents
        )
        
        # Assert all documents were created
        self.assertEqual(len(result), 3)
        
        # Verify count in collection
        count = self.collection.count_documents({})
        self.assertEqual(count, 3)

    def test_read_document(self):
        # Insert test document first
        inserted = self.collection.insert_one(self.test_data)
        
        # Test reading the document
        result = self.mongo_ops.read_document(
            collection_name='actions',
            query={"action_id": 1}
        )
        
        self.assertEqual(result["action_name"], "test_action")

    def test_read_many_documents(self):
        # Insert multiple test documents
        documents = [
            {"action_id": i, "status": "active"} 
            for i in range(3)
        ]
        self.collection.insert_many(documents)
        
        # Test reading with query
        results = self.mongo_ops.read_many_documents(
            collection_name='actions',
            query={"status": "active"}
        )
        
        self.assertEqual(len(list(results)), 3)

    def test_update_document(self):
        # Insert test document
        self.collection.insert_one(self.test_data)
        
        # Test updating
        update_result = self.mongo_ops.update_document(
            collection_name='actions',
            query={"action_id": 1},
            update={"$set": {"status": "inactive"}}
        )
        
        # Verify update
        updated_doc = self.collection.find_one({"action_id": 1})
        self.assertEqual(updated_doc["status"], "inactive")

    def test_delete_document(self):
        # Insert test document
        self.collection.insert_one(self.test_data)
        
        # Test deletion
        delete_result = self.mongo_ops.delete_document(
            collection_name='actions',
            query={"action_id": 1}
        )
        
        # Verify deletion
        self.assertEqual(self.collection.count_documents({}), 0)

    def test_aggregate_pipeline(self):
        # Insert test data
        documents = [
            {"action_id": i, "status": "active", "count": i} 
            for i in range(3)
        ]
        self.collection.insert_many(documents)
        
        # Test aggregation
        pipeline = [
            {"$match": {"status": "active"}},
            {"$group": {
                "_id": None,
                "total": {"$sum": "$count"}
            }}
        ]
        
        result = self.mongo_ops.aggregate(
            collection_name='actions',
            pipeline=pipeline
        )
        
        results_list = list(result)
        self.assertEqual(len(results_list), 1)
        self.assertEqual(results_list[0]["total"], 3)
```

## pytest
```python
import pytest
import mongomock
from datetime import datetime
from bson import ObjectId

# Import your MongoDB operations class
from database.mongo_operations import MongoOperations

@pytest.fixture
def mongo_ops():
    """Fixture for MongoDB operations with mongomock"""
    mock_client = mongomock.MongoClient()
    mock_db = mock_client['test_db']
    return MongoOperations(mock_db)

@pytest.fixture
def test_data():
    """Fixture for test data"""
    return {
        "action_id": 1,
        "action_name": "test_action",
        "created_time": datetime.utcnow(),
        "status": "active"
    }

@pytest.fixture
def collection(mongo_ops):
    """Fixture for MongoDB collection"""
    return mongo_ops.db['actions']

def test_create_document(mongo_ops, test_data, collection):
    # Test creating a single document
    result = mongo_ops.create_document(
        collection_name='actions',
        document=test_data
    )
    
    # Assert document was created
    assert result is not None
    
    # Verify document exists in collection
    saved_doc = collection.find_one({"action_id": 1})
    assert saved_doc["action_name"] == "test_action"

def test_create_many_documents(mongo_ops, collection):
    documents = [
        {"action_id": i, "action_name": f"test_{i}"} 
        for i in range(3)
    ]
    
    result = mongo_ops.create_many_documents(
        collection_name='actions',
        documents=documents
    )
    
    # Assert all documents were created
    assert len(result) == 3
    
    # Verify count in collection
    count = collection.count_documents({})
    assert count == 3

def test_read_document(mongo_ops, test_data, collection):
    # Insert test document first
    collection.insert_one(test_data)
    
    # Test reading the document
    result = mongo_ops.read_document(
        collection_name='actions',
        query={"action_id": 1}
    )
    
    assert result["action_name"] == "test_action"

def test_read_many_documents(mongo_ops, collection):
    # Insert multiple test documents
    documents = [
        {"action_id": i, "status": "active"} 
        for i in range(3)
    ]
    collection.insert_many(documents)
    
    # Test reading with query
    results = mongo_ops.read_many_documents(
        collection_name='actions',
        query={"status": "active"}
    )
    
    assert len(list(results)) == 3

def test_update_document(mongo_ops, test_data, collection):
    # Insert test document
    collection.insert_one(test_data)
    
    # Test updating
    update_result = mongo_ops.update_document(
        collection_name='actions',
        query={"action_id": 1},
        update={"$set": {"status": "inactive"}}
    )
    
    # Verify update
    updated_doc = collection.find_one({"action_id": 1})
    assert updated_doc["status"] == "inactive"

def test_delete_document(mongo_ops, test_data, collection):
    # Insert test document
    collection.insert_one(test_data)
    
    # Test deletion
    delete_result = mongo_ops.delete_document(
        collection_name='actions',
        query={"action_id": 1}
    )
    
    # Verify deletion
    assert collection.count_documents({}) == 0

def test_aggregate_pipeline(mongo_ops, collection):
    # Insert test data
    documents = [
        {"action_id": i, "status": "active", "count": i} 
        for i in range(3)
    ]
    collection.insert_many(documents)
    
    # Test aggregation
    pipeline = [
        {"$match": {"status": "active"}},
        {"$group": {
            "_id": None,
            "total": {"$sum": "$count"}
        }}
    ]
    
    result = mongo_ops.aggregate(
        collection_name='actions',
        pipeline=pipeline
    )
    
    results_list = list(result)
    assert len(results_list) == 1
    assert results_list[0]["total"] == 3
```

## pytest using mock instead of mongomock
```python
import pytest
from unittest.mock import Mock, MagicMock
from bson import ObjectId

@pytest.fixture
def mock_db():
    return MagicMock()

@pytest.fixture
def mongo_ops(mock_db):
    return MongoOperations(mock_db)

def test_create_document_with_mock(mongo_ops, mock_db):
    # Setup mock
    mock_collection = MagicMock()
    mock_db.__getitem__.return_value = mock_collection
    mock_collection.insert_one.return_value = Mock(
        inserted_id=ObjectId()
    )
    
    # Test
    document = {"action_id": 1, "name": "test"}
    result = mongo_ops.create_document('actions', document)
    
    # Verify
    mock_collection.insert_one.assert_called_once_with(document)
    assert result is not None

def test_read_with_error(mongo_ops, mock_db):
    # Setup mock to raise exception
    mock_collection = MagicMock()
    mock_db.__getitem__.return_value = mock_collection
    mock_collection.find_one.side_effect = Exception("Database error")
    
    # Test error handling
    with pytest.raises(Exception):
        mongo_ops.read_document('actions', {"id": 1})
```

## async
```python
import pytest
from motor.motor_asyncio import AsyncIOMotorClient
from unittest.mock import AsyncMock

@pytest.fixture
async def async_mongo_ops():
    mock_client = AsyncMock(spec=AsyncIOMotorClient)
    mock_db = mock_client['test_db']
    return MongoOperations(mock_db)

@pytest.mark.asyncio
async def test_async_create_document(async_mongo_ops):
    document = {"action_id": 1, "name": "test"}
    
    # Setup mock
    async_mongo_ops.db['actions'].insert_one = AsyncMock(
        return_value=Mock(inserted_id=ObjectId())
    )
    
    # Test
    result = await async_mongo_ops.create_document('actions', document)
    
    # Verify
    async_mongo_ops.db['actions'].insert_one.assert_called_once_with(document)
    assert result is not None
```

## mongo transaction
```python
import pytest
from unittest.mock import AsyncMock, MagicMock

@pytest.fixture
def mock_client():
    return MagicMock()

def test_transaction(mock_client, mongo_ops):
    # Mock session and transaction context managers
    mock_session = MagicMock()
    mock_session.__enter__.return_value = mock_session
    mock_session.__exit__.return_value = None
    
    mock_client.start_session.return_value = mock_session
    
    # Test transaction
    with mongo_ops.transaction() as session:
        mongo_ops.create_document(
            'actions',
            {"action_id": 1},
            session=session
        )
        mongo_ops.update_document(
            'related_actions',
            {"ref_id": 1},
            {"$set": {"status": "updated"}},
            session=session
        )
    
    # Verify transaction was committed
    mock_session.commit_transaction.assert_called_once()
```

## Remember to

Remember to:
Use fixtures for common setup
Test both success and error cases
Mock external dependencies when needed
Use parametrize for multiple test cases
Clean up test data between tests