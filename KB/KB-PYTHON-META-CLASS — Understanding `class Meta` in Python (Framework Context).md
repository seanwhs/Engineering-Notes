# KB-PYTHON-META-CLASS — Understanding `class Meta` in Python (Framework Context)

**Knowledge Base Category:** Python / Frameworks / Django / DRF
**System:** Django, Django REST Framework, Python
**Severity:** Low-Medium (Conceptual)
**Status:** Active Reference

---

## 1. Purpose

This note explains the use of `class Meta` in Python, specifically in frameworks like Django and DRF, as a **pattern for providing metadata to classes**. It clarifies how `Meta` works, why it's used, and common options.

---

## 2. Overview

* `class Meta` is an **inner class** of another class.
* Frameworks read the `Meta` class to **configure behavior or options**.
* It does **not change Python itself**; it’s a framework convention.

```
OuterClass
 ├── attributes / methods
 └── class Meta
        ├── framework options
        └── metadata
```

---

## 3. Examples

### 3.1 Django Model

```python
from django.db import models

class Book(models.Model):
    title = models.CharField(max_length=200)
    author = models.CharField(max_length=100)

    class Meta:
        ordering = ['title']        # Default queryset ordering
        verbose_name = 'Book'       # Singular display name
        verbose_name_plural = 'Books'
```

### 3.2 Django Form

```python
from django import forms
from myapp.models import Book

class BookForm(forms.ModelForm):
    class Meta:
        model = Book
        fields = ['title', 'author']
```

### 3.3 DRF Serializer

```python
from rest_framework import serializers

class BookSerializer(serializers.ModelSerializer):
    class Meta:
        model = Book
        fields = ['title', 'author']
```

---

## 4. Key Points

* `Meta` is **optional**, but widely used for configuration.
* Frameworks inspect `Meta` at runtime to adjust behavior.
* Typical use-cases:

  * Model options: `ordering`, `db_table`, `unique_together`
  * Form/Serializer options: `model`, `fields`, `read_only_fields`
* Frameworks do **not require the inner class name to be Meta**, but the convention is strong.

---

## 5. Diagram

```
+--------------------+
|      Book          | <- Outer Class
|--------------------|
| title              |
| author             |
| methods            |
|--------------------|
| + Meta             | <- Inner class for metadata
|   - ordering       |
|   - verbose_name   |
|   - verbose_name_plural |
+--------------------+
```

* Outer class: main model/form/serializer
* Inner `Meta` class: configuration / options read by framework

---

## 6. Summary

* `class Meta` is a **framework convention**, not a Python feature.
* Provides **metadata and configuration** to outer classes.
* Widely used in Django models, forms, and DRF serializers.
* Helps avoid hardcoding behavior in methods, centralizing **config**.

---

**Owner:** Platform / Python Engineering
**Last Reviewed:** 2025-12-23
