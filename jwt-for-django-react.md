# **The Complete Guide: Implementing JWT in Django REST Framework & React**

This guide provides a professional-grade implementation of **Stateless Authentication** using `djangorestframework-simplejwt` on the backend and **React (Vite) + Axios** on the frontend. This setup includes automated token refreshing and **Protected Routing** to ensure a secure, seamless user experience.

---

## **1. The Mental Model**

In a JWT-based system, the server does not store session data. It issues a signed token to the client, which the server verifies cryptographically. This allows for **Horizontal Scalability**, as any server instance can verify the user without a shared session store.

---

## **2. Installation & Backend Setup**

### **2.1 Required Packages**

Install the JWT library and `django-cors-headers` to allow your React app (port 5173) to communicate with your Django API.

```bash
pip install djangorestframework-simplejwt django-cors-headers

```

### **2.2 Django Settings (`settings.py`)**

```python
INSTALLED_APPS = [
    ...
    'corsheaders',
    'rest_framework',
    'rest_framework_simplejwt.token_blacklist', 
]

MIDDLEWARE = [
    'corsheaders.middleware.CorsMiddleware', 
    'django.middleware.common.CommonMiddleware',
    ...
]

CORS_ALLOWED_ORIGINS = ["http://localhost:5173"]

REST_FRAMEWORK = {
    'DEFAULT_AUTHENTICATION_CLASSES': (
        'rest_framework_simplejwt.authentication.JWTAuthentication',
    )
}

SIMPLE_JWT = {
    'ACCESS_TOKEN_LIFETIME': timedelta(minutes=15),
    'REFRESH_TOKEN_LIFETIME': timedelta(days=50),
    'ROTATE_REFRESH_TOKENS': True,
    'BLACKLIST_AFTER_ROTATION': True,
    'AUTH_HEADER_TYPES': ('Bearer',),
}

```

---

## **3. Custom Views & Testing**

### **3.1 Auth Views (`views.py`)**

```python
from rest_framework_simplejwt.views import TokenObtainPairView
from rest_framework_simplejwt.tokens import RefreshToken
from rest_framework.views import APIView
from rest_framework.response import Response
from rest_framework.permissions import IsAuthenticated
from rest_framework import status

class LogoutView(APIView):
    permission_classes = [IsAuthenticated]
    def post(self, request):
        try:
            refresh_token = request.data["refresh"]
            token = RefreshToken(refresh_token)
            token.blacklist()
            return Response(status=status.HTTP_205_RESET_CONTENT)
        except Exception:
            return Response(status=status.HTTP_400_BAD_REQUEST)

```

### **3.2 Django Integration Test (`tests.py`)**

```python
from django.urls import reverse
from django.contrib.auth.models import User
from rest_framework.test import APITestCase
from rest_framework import status

class AuthTests(APITestCase):
    def test_login_and_logout(self):
        User.objects.create_user(username="test", password="password123")
        # Login
        res = self.client.post(reverse('token_obtain_pair'), {"username": "test", "password": "password123"})
        access = res.data['access']
        # Logout
        self.client.credentials(HTTP_AUTHORIZATION=f'Bearer {access}')
        logout_res = self.client.post(reverse('auth_logout'), {"refresh": res.data['refresh']})
        self.assertEqual(logout_res.status_code, status.HTTP_205_RESET_CONTENT)

```

---

## **4. Frontend Implementation: React & Axios**

### **4.1 Axios Interceptor (`api.js`)**

The interceptor handles 401 errors by refreshing the token and retrying the original request automatically.

```javascript
import axios from 'axios';

const api = axios.create({ baseURL: 'http://localhost:8000/api' });

api.interceptors.request.use((config) => {
    const token = localStorage.getItem('access_token');
    if (token) config.headers.Authorization = `Bearer ${token}`;
    return config;
});

api.interceptors.response.use(
    (res) => res,
    async (error) => {
        const originalRequest = error.config;
        if (error.response.status === 401 && !originalRequest._retry) {
            originalRequest._retry = true;
            try {
                const refresh = localStorage.getItem('refresh_token');
                const { data } = await axios.post('http://localhost:8000/api/token/refresh/', { refresh });
                localStorage.setItem('access_token', data.access);
                return api(originalRequest);
            } catch (err) {
                localStorage.clear();
                window.location.href = '/login';
            }
        }
        return Promise.reject(error);
    }
);
export default api;

```

---

## **5. React Router: Protected Routes**

### **5.1 ProtectedRoute Component**

Create a wrapper that checks for the existence of an access token before rendering children.

```jsx
import { Navigate } from 'react-router-dom';

export default function ProtectedRoute({ children }) {
    const token = localStorage.getItem('access_token');
    return token ? children : <Navigate to="/login" />;
}

```

### **5.2 Main Routing (`App.jsx`)**

```jsx
import { BrowserRouter, Routes, Route } from 'react-router-dom';
import Login from './Login';
import Dashboard from './Dashboard';
import ProtectedRoute from './ProtectedRoute';

function App() {
    return (
        <BrowserRouter>
            <Routes>
                <Route path="/login" element={<Login />} />
                <Route 
                    path="/dashboard" 
                    element={
                        <ProtectedRoute>
                            <Dashboard />
                        </ProtectedRoute>
                    } 
                />
            </Routes>
        </BrowserRouter>
    );
}

```

---

## **6. Engineering Trade-offs**

| Setting | Security Posture | Trade-off |
| --- | --- | --- |
| **Protected Routes** | **Medium** | Client-side check only; API still requires token verification. |
| **Axios Interceptor** | **Excellent UX** | Completely hides token refresh logic from components. |
| **LocalStorage** | **Vulnerable** | Susceptible to XSS. Use **HttpOnly Cookies** for high-security environments. |

---

## **7. Professional Best Practices**

1. **Migrations:** Run `python manage.py migrate` for the blacklist tables.
2. **Double Verification:** Never rely *only* on React Router for security; the Backend is the ultimate gatekeeper.
3. **Use HTTPS:** In production, JWTs must never travel over unencrypted HTTP.

