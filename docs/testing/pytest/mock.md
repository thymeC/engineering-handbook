Mock for Unit Test
Remember to use the appropriate mock type based on your testing needs:
- Use Mock when you want to be strict about the interface
- Use MagicMock when you need more flexibility
- Use AsyncMock for async functions
- Use patch for replacing objects in your code during testing

```python
from unittest import TestCase
from unittest.mock import Mock, MagicMock, patch
from utils.data_mapper import DataMapper

class TestDataMapper(TestCase):
    def setUp(self):
        self.mapper = DataMapper()
        
    def test_map_record_with_mock(self):
        # Mock is more strict - you need to specify what to mock
        mock_source = Mock()
        mock_source.get.return_value = "test_value"
        
        # This would raise AttributeError because Mock doesn't create attributes automatically
        # print(mock_source.undefined_attribute)
        
        # Need to explicitly set attributes
        mock_source.id = 1
        mock_source.name = "test"
        
        result = self.mapper.map_record(mock_source)
        self.assertEqual(result["action_id"], 1)

    def test_map_record_with_magic_mock(self):
        # MagicMock automatically creates attributes and methods
        magic_mock_source = MagicMock()
        
        # These work without explicit definition
        magic_mock_source.id = 1
        magic_mock_source.undefined_method()  # This works!
        print(magic_mock_source.undefined_attribute)  # This also works!
        
        result = self.mapper.map_record(magic_mock_source)
        self.assertEqual(result["action_id"], 1)

    @patch('utils.data_mapper.PipelineInit')
    def test_pipeline_inheritance(self, mock_pipeline):
        # Test pipeline initialization
        mapper = DataMapper()
        mock_pipeline.assert_called_once()
        
        # Verify metadata was reset
        self.assertIn("pipeline_name", mapper.meta_data)

    def test_complete_mapping(self):
        # Test with complete mock data
        mock_data = {
            "id": 1,
            "name": "test",
            "status": "active",
            "product": "product_a",
            "category": "cat_1",
            "created_at": "2024-01-01",
            "updated_at": "2024-01-02"
        }
        
        expected = {
            "action_id": 1,
            "action_name": "test",
            "action_status": "active",
            "product": "product_a",
            "category": "cat_1",
            "created_time": "2024-01-01",
            "updated_time": "2024-01-02"
        }
        
        result = self.mapper.map_record(mock_data)
        self.assertEqual(result, expected)

    def test_missing_fields(self):
        # Test with missing fields
        mock_data = {
            "id": 1,
            "name": "test"
        }
        
        result = self.mapper.map_record(mock_data)
        
        # Verify only mapped fields are present
        self.assertEqual(result["action_id"], 1)
        self.assertEqual(result["action_name"], "test")
        self.assertNotIn("action_status", result)

    @patch('utils.data_mapper.DataMapper.map_record')
    def test_method_calls(self, mock_map_record):
        # Setup mock return value
        mock_map_record.return_value = {"action_id": 1}
        
        # Call the method
        self.mapper.map_record({"id": 1})
        
        # Verify the method was called with correct arguments
        mock_map_record.assert_called_once_with({"id": 1})
```

## Key differences between Mock and MagicMock:
### Mock:
- More strict
- Doesn't create attributes automatically
- Better for enforcing interface contracts
- Use when you want to be explicit about what's available
```python
# Example with Mock
mock = Mock()
mock.defined_method.return_value = "test"  # Need to define first
mock.defined_method()  # Works
mock.undefined_method()  # Raises AttributeError
```
### MagicMock
- More permissive
- Automatically creates attributes and methods
- Has default implementations for magic methods
- Better for rapid prototyping or less strict testing
```python
# Example with MagicMock
magic_mock = MagicMock()
magic_mock.undefined_method()  # Works automatically
magic_mock.undefined_attribute  # Returns another MagicMock
```

### An example with context managers and async:
Note: Not quite understand this part
```python
from unittest import IsolatedAsyncioTestCase
from unittest.mock import AsyncMock, patch
from contextlib import asynccontextmanager

class TestDataMapperAdvanced(IsolatedAsyncioTestCase):
    async def asyncSetUp(self):
        self.mapper = DataMapper()

    @patch('utils.data_mapper.some_async_function')
    async def test_async_mapping(self, mock_async_func):
        # Setup async mock
        mock_async_func.return_value = {"id": 1}
        
        result = await self.mapper.async_map_record({"id": 1})
        self.assertEqual(result["action_id"], 1)

    async def test_with_context_manager(self):
        # Mock a context manager
        @asynccontextmanager
        async def mock_connection():
            try:
                yield {"id": 1}
            finally:
                pass

        with patch('utils.data_mapper.get_connection', mock_connection):
            async with self.mapper.get_connection() as data:
                result = await self.mapper.async_map_record(data)
                self.assertEqual(result["action_id"], 1)

    def test_mock_chain(self):
        # Mocking chained methods
        mock = MagicMock()
        mock.method1.return_value.method2.return_value = "test"
        
        # This works
        result = mock.method1().method2()
        self.assertEqual(result, "test")
```
Best practices for mocking:
1. Use Mock for Strict Testing:
```python
def test_strict_interface():
    mock = Mock(spec=YourClass)  # Only allows defined methods
    with self.assertRaises(AttributeError):
        mock.undefined_method()
```
2. Use MagicMock for Flexibility:
```python
def test_flexible_mocking():
    mock = MagicMock()
    # No need to define methods/attributes
    mock.any_method()
    mock.any_attribute
```
3. Use patch Decorator/Context Manager:
```python
@patch('module.ClassName')
def test_with_patch(mock_class):
    mock_class.return_value.method.return_value = "test"
    
# Or with context manager
def test_with_context():
    with patch('module.ClassName') as mock_class:
        mock_class.return_value.method.return_value = "test"
```
4. Mock Side Effects:
```python
def test_side_effects():
    mock = Mock()
    mock.method.side_effect = [1, 2, 3]  # Returns values in sequence
    mock.method()  # Returns 1
    mock.method()  # Returns 2
```
5. Assert Call Arguments:
```python
def test_call_arguments():
    mock = Mock()
    mock.method(1, key="value")
    mock.method.assert_called_with(1, key="value")
    mock.method.assert_called_once()
```

## Conclusion
Remember to use the appropriate mock type based on your testing needs:
- Use Mock when you want to be strict about the interface
- Use MagicMock when you need more flexibility
- Use AsyncMock for async functions
- Use patch for replacing objects in your code during testing
