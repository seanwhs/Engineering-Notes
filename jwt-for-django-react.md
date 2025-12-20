# **Guide to JWT Authentication with Django & React**

This guide provides a professional-grade implementation of **Stateless Authentication** using `djangorestframework-simplejwt` and **React (Vite) + Axios**. This setup covers the full lifecycle: secure backend configuration, custom claims, automated token refreshing (interceptors), and client-side protected routing.

---

## **1. The Architectural Mental Model**

In a JWT-based system, the server is "blind" to sessions. It doesn't store login states in memory. Instead, it issues a signed token to the client. The client presents this token in the `Authorization` header, and the server verifies it cryptographically using a secret key. This enables **Horizontal Scalability**, as any server instance can verify any user without a shared session database.

---

## **2. Backend Implementation (Django)**

### **2.1 Installation**

Install the core JWT library and CORS headers to allow your React app (port 5173) to communicate with your Django API.

```bash
pip install djangorestframework-simplejwt django-cors-headers

```

### **2.2 Configuration (`settings.py`)**

```python
from datetime import timedelta

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
    'UPDATE_LAST_LOGIN': False,

    'ALGORITHM': 'HS256',
    'VERIFYING_KEY': None,
    'AUDIENCE': None,
    'ISSUER': None,
    'JWK_URL': None,
    'LEEWAY': 0,

    'AUTH_HEADER_TYPES': ('Bearer',),
    'AUTH_HEADER_NAME': 'HTTP_AUTHORIZATION',
    'USER_ID_FIELD': 'id',
    'USER_ID_CLAIM': 'user_id',
    'USER_AUTHENTICATION_RULE': 'rest_framework_simplejwt.authentication.default_user_authentication_rule',

    'AUTH_TOKEN_CLASSES': ('rest_framework_simplejwt.tokens.AccessToken',),
    'TOKEN_TYPE_CLAIM': 'token_type',
    'TOKEN_USER_CLASS': 'rest_framework_simplejwt.models.TokenUser',

    'JTI_CLAIM': 'jti',

    'SLIDING_TOKEN_REFRESH_EXP_CLAIM': 'refresh_exp',
    'SLIDING_TOKEN_LIFETIME': timedelta(minutes=5),
    'SLIDING_TOKEN_REFRESH_LIFETIME': timedelta(days=1),
}

```

---

## **3. Custom Logic & Testing**

### **3.1 Custom Claims (`serializers.py`)**

Extend the JWT payload to include user data (like `username` or `email`) so the frontend can personalize the UI without a secondary API call.

```python
from rest_framework_simplejwt.serializers import TokenObtainPairSerializer

class MyTokenObtainPairSerializer(TokenObtainPairSerializer):
    @classmethod
    def get_token(cls, user):
        token = super().get_token(user)
        # Custom claims
        token['username'] = user.username
        token['email'] = user.email
        return token

```

### **3.2 Logout View (`views.py`)**

Blacklisting the refresh token ensures the session is invalidated on the server side.

```python
from rest_framework_simplejwt.tokens import RefreshToken
from rest_framework.views import APIView
from rest_framework.response import Response
from rest_framework import status
from rest_framework.permissions import IsAuthenticated

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

### **3.3 Django Framework Integration Test (`tests.py`)**

```python
from django.urls import reverse
from django.contrib.auth.models import User
from rest_framework.test import APITestCase
from rest_framework import status

class AuthTests(APITestCase):
    def setUp(self):
        self.user = User.objects.create_user(username="test", password="password123")

    def test_full_auth_cycle(self):
        # 1. Login
        res = self.client.post(reverse('token_obtain_pair'), {"username": "test", "password": "password123"})
        self.assertEqual(res.status_code, status.HTTP_200_OK)
        access, refresh = res.data['access'], res.data['refresh']

        # 2. Logout
        self.client.credentials(HTTP_AUTHORIZATION=f'Bearer {access}')
        logout_res = self.client.post(reverse('auth_logout'), {"refresh": refresh})
        self.assertEqual(logout_res.status_code, status.HTTP_205_RESET_CONTENT)

```

---

## **4. Frontend Implementation (React + Axios)**

### **4.1 Axios Interceptor (`api.js`)**

The "Interceptor" automatically handles 401 errors by attempting a token refresh and retrying the failed request.

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
                const { data } = await axios.post('/token/refresh/', { refresh });
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

### **5.1 Route Guard (`ProtectedRoute.jsx`)**

Prevents unauthenticated users from accessing specific components.

```jsx
import { Navigate } from 'react-router-dom';

export default function ProtectedRoute({ children }) {
    const token = localStorage.getItem('access_token');
    return token ? children : <Navigate to="/login" />;
}

```

---

## **6. Engineering Trade-offs**

| Setting | Posture | Trade-off |
| --- | --- | --- |
| **Token Rotation** | **High Security** | Every refresh issues a new refresh token, preventing replay attacks. |
| **Custom Claims** | **Better UX** | Eliminates extra "fetch user profile" calls on initial load. |
| **LocalStorage** | **Low Security** | Vulnerable to XSS; for highly sensitive data, use **HttpOnly Cookies**. |

---

## **7. Professional Checklist**

1. **Migrations:** Ensure `python manage.py migrate` is run for the blacklist tables.
2. **Environment Variables:** Always keep your `SECRET_KEY` and allowed origins out of source control.
3. **HTTPS:** JWTs are essentially "passwords" in a header; they must be encrypted in transit via SSL.

