# **Engineering Note: Django Decorators**

## **Project Context**

In Django, a **decorator** is a **Python function** that wraps another function (or method) to modify its behavior without changing its code. Decorators are widely used to **add cross-cutting concerns** like authentication, permission checks, caching, logging, and rate-limiting to **function-based views (FBVs)** or methods in classes.

> **Key idea:** Decorators = reusable function wrappers. They enhance functions without modifying the function’s core logic.

---

## **1. Overview of Decorators**

A **decorator** is a higher-order function:

* It **takes a function as input**.
* It **returns a new function** with added behavior.
* It allows **code reuse and separation of concerns**.

**Python example:**

```python
def debug(func):
    def wrapper(*args, **kwargs):
        print(f"Calling {func.__name__}...")
        result = func(*args, **kwargs)
        print(f"{func.__name__} returned {result}")
        return result
    return wrapper

@debug
def add(a, b):
    return a + b

add(2, 3)
# Output:
# Calling add...
# add returned 5
```

> The `@debug` decorator wraps the `add` function, adding logging behavior.

---

## **2. Why Use Decorators in Django**

### **2.1 Code Reuse and DRY**

Without decorators, adding common functionality requires repeating code:

```python
def my_view(request):
    if not request.user.is_authenticated:
        return redirect('login')
```

* Multiple views would need the same check.
* A decorator allows **centralizing this logic**, keeping views DRY.

---

### **2.2 Separation of Concerns**

Decorators allow you to **separate auxiliary logic** (authentication, permissions, caching) from **core business logic** in views.

```python
from django.contrib.auth.decorators import login_required

@login_required
def my_view(request):
    # Core logic
    ...
```

* The view focuses only on what it’s supposed to do.
* Authentication is handled outside the view.

---

### **2.3 Flexibility and Composability**

Multiple decorators can be **stacked**:

```python
from django.views.decorators.cache import cache_page
from django.contrib.auth.decorators import login_required

@login_required
@cache_page(60 * 15)
def my_view(request):
    ...
```

* The request is cached for 15 minutes **and** requires login.
* Each decorator addresses a **single concern**, making code modular.

---

## **3. Common Django Decorators**

| Decorator                      | Purpose                            | Module                           |
| ------------------------------ | ---------------------------------- | -------------------------------- |
| `@login_required`              | Require authenticated user         | `django.contrib.auth.decorators` |
| `@permission_required('perm')` | Require specific permission        | `django.contrib.auth.decorators` |
| `@user_passes_test(func)`      | Custom test function               | `django.contrib.auth.decorators` |
| `@require_http_methods([...])` | Allow only specified HTTP methods  | `django.views.decorators.http`   |
| `@cache_page(seconds)`         | Cache response for a given time    | `django.views.decorators.cache`  |
| `@csrf_exempt`                 | Exempt a view from CSRF protection | `django.views.decorators.csrf`   |
| `@vary_on_cookie`              | Vary cache based on cookies        | `django.views.decorators.vary`   |

---

## **4. Examples**

### **4.1 Authentication with `login_required`**

```python
from django.shortcuts import render
from django.contrib.auth.decorators import login_required

@login_required(login_url='/auth/login/')
def dashboard(request):
    return render(request, 'dashboard.html')
```

> Only logged-in users can access the dashboard. Unauthenticated users are redirected to `/auth/login/`.

---

### **4.2 Custom Decorator**

```python
from functools import wraps
from django.http import HttpResponseForbidden

def staff_required(view_func):
    @wraps(view_func)
    def _wrapped_view(request, *args, **kwargs):
        if not request.user.is_staff:
            return HttpResponseForbidden("Staff only")
        return view_func(request, *args, **kwargs)
    return _wrapped_view

@staff_required
def admin_dashboard(request):
    ...
```

> This decorator checks if the user is staff before executing the view.

---

### **4.3 Composing Multiple Decorators**

```python
from django.views.decorators.cache import cache_page
from django.contrib.auth.decorators import login_required

@login_required
@cache_page(60 * 5)
def profile(request):
    ...
```

* Login is required **and** the page is cached for 5 minutes.
* Order matters: decorators are applied from **bottom to top**.

---

## **5. Decorators vs Mixins**

| Feature        | FBV        | CBV    |
| -------------- | ---------- | ------ |
| Reuse behavior | Decorators | Mixins |
| Inheritance    | N/A        | Yes    |
| Extensibility  | Moderate   | High   |
| Typical Use    | FBVs       | CBVs   |

* **FBVs:** Use decorators to add reusable behavior.
* **CBVs:** Use mixins instead; decorators can also wrap CBV methods (`method_decorator`).

```python
from django.utils.decorators import method_decorator
from django.contrib.auth.decorators import login_required
from django.views import View

@method_decorator(login_required, name='dispatch')
class DashboardView(View):
    ...
```

---

## **6. Best Practices**

1. Keep decorators **focused on a single concern**.
2. Use `functools.wraps` to preserve function metadata.
3. Stack decorators carefully; understand the execution order.
4. For CBVs, use `method_decorator` to wrap class methods.
5. Test decorators independently to ensure they don’t break core logic.

---

## **7. Summary**

* **Purpose:** Decorators provide **reusable, modular, and composable functionality** for FBVs.
* **Benefits:** DRY, clean, testable, and maintainable code.
* **Best Use Case:** FBVs and method-level CBV behavior.
* **Key Difference from Mixins:** Mixins are class-based; decorators are function-based.

> Django decorators are a **powerful tool** to separate concerns, enforce policies, and extend functionality without cluttering view logic.

---


Do you want me to do that?
