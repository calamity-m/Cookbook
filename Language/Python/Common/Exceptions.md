# Custom Exceptions

```python
# Simple exception case
class MyCustomException(Exception): 
	pass

# Contextual Exception

class ContextualException:
	def __init__(self, context):
		self.context = context

	def __str__(self):
		return(f"Contextual Exception Occured: {self.context}")	