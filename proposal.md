# Proposal: Introducing `@super_init` Decorator in Django  

## **Summary**  
In Django, class-based inheritance requires using `super().__init__()` to call the parent class constructor. However, this can lead to **unnecessary executions, complexity in multi-level inheritance, and limitations on heavy operations** (e.g., API calls).  

This proposal introduces a **`@super_init` decorator** to provide **optional parent class initialization**, improving flexibility and readability.

---

## **Current Approach & Its Limitations**  

Currently, Django follows the standard approach where child classes must explicitly call `super().__init__()` to initialize the parent class.

### **Example (Current Approach)**
```python
class Parent:
    def __init__(self):
        print("Parent init")

class Child(Parent):
    def __init__(self):
        super().__init__()  # Always runs parent init
        print("Child init")

c = Child()
# Output: 
# Parent init
# Child init
```

### **Issues with This Approach:**  
- 🔴 **Always Executes Parent Init:** No option to **skip** it when unnecessary.  
- 🔴 **Performance Overhead:** Even when parent initialization is not needed, `super().__init__()` is still executed.  
- 🔴 **Complexity in Multi-Level Inheritance:** Each level in inheritance must handle `super()` correctly, increasing potential errors.  
- 🔴 **Difficult to Use with API Calls:** API calls or expensive computations **cannot be safely placed** inside `__init__`, forcing developers to separate them.  

---

## **Proposed Solution: `@super_init` Decorator**  
We introduce a **decorator-based approach** that allows **optional execution** of the parent `__init__` method, improving **code clarity, flexibility, and maintainability**.

### **Decorator Implementation**
```python
def super_init(cls):
    """Decorator to optionally initialize the parent class."""
    def wrapper(self, *args, **kwargs):
        if kwargs.get("use_parent_init", True):
            super(cls, self).__init__(*args, **kwargs)
    return wrapper
```

### **How It Works**
1. If `use_parent_init=True` (default), the parent’s `__init__` runs.  
2. If `use_parent_init=False`, the parent’s `__init__` is skipped, reducing unnecessary executions.  

---

## **Example Usage (Improved Approach)**  

### **Before (Current Way)**
```python
class Parent:
    def __init__(self):
        print("Parent init")

class Child(Parent):
    def __init__(self):
        super().__init__()  # Always runs parent init
        print("Child init")

c = Child()
# Output:
# Parent init
# Child init
```

### **After (Using `@super_init`)**
```python
class Parent:
    def __init__(self):
        print("Parent init")

class Child(Parent):
    @super_init
    def __init__(self):
        print("Child init")

c1 = Child()  # No parent init
c2 = Child(use_parent_init=True)  # Runs parent init

# Output: 
# Child init
# Parent init → Child init (if `use_parent_init=True`)
```

---

## **Multi-Level Inheritance & Its Complexity Without `@super_init`**
### **Example (Traditional Multi-Level Inheritance)**
```python
class A:
    def __init__(self):
        print("Init A")

class B(A):
    def __init__(self):
        super().__init__()
        print("Init B")

class C(B):
    def __init__(self):
        super().__init__()
        print("Init C")

c = C()
# Output:
# Init A
# Init B
# Init C
```
🔴 Every class in the chain **must** call `super().__init__()`, even if it's unnecessary.  

### **Solution with `@super_init`**
```python
class A:
    def __init__(self):
        print("Init A")

class B(A):
    @super_init
    def __init__(self):
        print("Init B")

class C(B):
    @super_init
    def __init__(self):
        print("Init C")

c1 = C()  # No parent init
c2 = C(use_parent_init=True)  # Runs full inheritance chain

# Output:
# Init C
# Init B → Init C (if `use_parent_init=True`)
# Init A → Init B → Init C (if `use_parent_init=True` in B and C)
```
✅ **Each class can decide whether to initialize its parent dynamically.**  
✅ **Simplifies multi-level inheritance & removes unnecessary method calls.**  

---

## **Key Benefits of `@super_init`**
✅ **More Control Over Parent Initialization** – Child classes can now decide when to initialize the parent.  
✅ **Better Performance** – Avoid unnecessary method calls and overhead.  
✅ **Improved Readability & Maintainability** – Reduces code clutter, making inheritance clearer.  
✅ **Solves API Call Issues in `__init__`** – Allows API calls inside `__init__` without forcing execution in every instance.  
✅ **Simplifies Multi-Level Inheritance** – No need to handle complex `super()` chains manually.  

---

## **Potential Challenges & Counterarguments**  

| **Challenge** | **Solution** |
|--------------|-------------|
| **Backward Compatibility** – Existing Django classes use `super()` | `@super_init` is **optional**, allowing gradual adoption. |
| **Developer Learning Curve** | A **clear Django doc update** will explain its advantages. |
| **Metaclass Interactions** | Needs testing with Django’s metaclass-based class structures. |

---

## **Impact on Django’s Future**  
🔹 Makes Django **more Pythonic and developer-friendly**.  
🔹 Enhances **flexibility in class-based views, models, and middleware**.  
🔹 Optimizes performance in scenarios where unnecessary parent init execution adds overhead.  

This decorator could become **a standard practice in Django for class-based development**, just like `@classmethod` and `@staticmethod`.

---

## **Next Steps**  
1️⃣ **Community Feedback** – Open discussion on whether this should be added to Django core.  
2️⃣ **Testing** – Conduct compatibility testing with Django’s existing class-based structures.  
3️⃣ **Implementation** – If accepted, contribute a PR to integrate `@super_init` in Django core.

---

## **Conclusion**  
`@super_init` offers a **cleaner, more efficient, and flexible** way to handle class-based inheritance in Django. It simplifies code, reduces performance overhead, and solves real-world problems in Django development.

🔹 **Let’s discuss and refine this idea together!** 🚀
