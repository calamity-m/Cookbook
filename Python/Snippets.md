[[Dreambook/Void/Snippets|Snippets]]

# Hello World

```python
def hello_world():
	return "Hello World!"

if __name__ == '__main__':
	print(f"Woaaah!? {hello_world()}")
```


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
```
