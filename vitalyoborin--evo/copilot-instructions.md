## evo

> Open/Closed Principle (OCP)


# Open/Closed Principle (OCP) in Python

**Definition**:

> *A class, function, or module should be open for extension but closed for modification.*

That means you should be able to **add new functionality without changing existing code**.

---

## 📌 1. Classes

❌ Bad example (every time we add a new shape, we must edit `AreaCalculator`):

```python
class Rectangle:
    def __init__(self, w, h):
        self.w, self.h = w, h

class Circle:
    def __init__(self, r):
        self.r = r

class AreaCalculator:
    def area(self, shape):
        if isinstance(shape, Rectangle):
            return shape.w * shape.h
        elif isinstance(shape, Circle):
            return 3.14 * shape.r**2
```

✅ Good example (extensible via polymorphism):

```python
class Shape:
    def area(self):
        raise NotImplementedError

class Rectangle(Shape):
    def __init__(self, w, h):
        self.w, self.h = w, h

    def area(self):
        return self.w * self.h

class Circle(Shape):
    def __init__(self, r):
        self.r = r

    def area(self):
        return 3.14 * self.r**2

# Adding a new shape does NOT require modifying AreaCalculator
class Triangle(Shape):
    def __init__(self, base, h):
        self.base, self.h = base, h

    def area(self):
        return 0.5 * self.base * self.h

def total_area(shapes):
    return sum(shape.area() for shape in shapes)
```

---

## 📌 2. Functions

❌ Bad example (logic is hardcoded):

```python
def discount(price, type):
    if type == "student":
        return price * 0.9
    elif type == "vip":
        return price * 0.8
```

✅ Good example (new discounts can be added without touching existing code):

```python
class Discount:
    def apply(self, price): ...

class StudentDiscount(Discount):
    def apply(self, price): return price * 0.9

class VIPDiscount(Discount):
    def apply(self, price): return price * 0.8

def apply_discount(price, strategy: Discount):
    return strategy.apply(price)
```

---

## 📌 3. Modules & Pipelines (AI/ML Example)

❌ Bad example:

```python
def evaluate(model, X, y, metric="accuracy"):
    if metric == "accuracy":
        return accuracy_score(y, model.predict(X))
    elif metric == "f1":
        return f1_score(y, model.predict(X))
```

✅ Good example (extensible metrics):

```python
class Metric:
    def calculate(self, y_true, y_pred): ...

class Accuracy(Metric):
    def calculate(self, y_true, y_pred):
        return (y_true == y_pred).mean()

class F1Score(Metric):
    def calculate(self, y_true, y_pred): ...

def evaluate(model, X, y, metric: Metric):
    return metric.calculate(y, model.predict(X))
```

Adding a new metric = implement a new class.
No modification to `evaluate` required.

---

## 📌 4. Dependency Injection

OCP works best with **dependency injection**:

* The **high-level module** depends on abstractions (interfaces)
* New implementations can be injected at runtime

✅ Example:

```python
class Trainer:
    def __init__(self, model):
        self.model = model  # model can be Logistic, RandomForest, NeuralNet

    def train(self, X, y):
        self.model.fit(X, y)
```

---

## 📌 5. Configuration-driven Extensibility

Instead of hardcoding, use configs:

```yaml
model: "RandomForest"
metrics:
  - "Accuracy"
  - "F1Score"
```

Your Python code just loads implementations by name → OCP in action.

---

# ✅ Summary

* **Encapsulate variation** in interfaces or abstract classes
* **Extend via new subclasses** instead of editing existing logic
* **Use dependency injection/configuration** to swap behaviors
* In AI/ML: OCP helps when you want to try new models, metrics, or preprocessing steps without breaking the pipeline

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/VitalyOborin)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/VitalyOborin)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
