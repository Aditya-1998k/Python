# 🧭 Django 50
## 1. Django Core Concepts (Basics) — 5 Questions
1. What is Django and explain its MVT architecture?  
2. How does Django handle a request internally (Request–Response cycle)?  
3. What are Django projects and apps, and why separate them?  
4. What is the role of `manage.py` and `settings.py`?  
5. How is Django different from Flask or FastAPI?  

---

## 2. Models & ORM — 6 Questions
1. What is a Django model and how do migrations work?  
2. What are QuerySets and how are they evaluated?  
3. Explain `select_related()` vs `prefetch_related()`.  
4. How to perform CRUD operations using the ORM?  
5. What are custom managers and querysets?  
6. How to handle transactions and optimize ORM queries?  

---

## 3. Views & URL Routing — 4 Questions
1. What’s the difference between function-based and class-based views?  
2. What are Django generic views and why use them?  
3. How does URL routing work and what is `reverse()` used for?  
4. How do you handle custom 404 or 500 error pages?  

---

## 4. Templates & Forms — 4 Questions
1. How does Django’s template engine work?  
2. What are template tags, filters, and context processors?  
3. How does form validation work and what is CSRF protection?  
4. Difference between `forms.Form` and `forms.ModelForm`.  

---

## 5. Authentication & Authorization — 4 Questions
1. How does Django’s authentication system work?  
2. How do you create a custom user model?  
3. What are groups and permissions in Django?  
4. How are sessions managed in Django?  

---

## 6. Middlewares & Signals — 3 Questions
1. What is middleware and how does it work internally?  
2. How to create custom middleware and what are use cases?  
3. What are Django signals and when should you use or avoid them?  

---

## 7. Admin Customization — 2 Questions
1. How to register and customize models in Django admin?  
2. What are `list_display`, `list_filter`, and custom admin actions?  

---

## 8. Django REST Framework (DRF) — 6 Questions
1. What is DRF and how is it different from Django views?  
2. Explain Serializers and ModelSerializers.  
3. Difference between `APIView`, `GenericViewSet`, and `ViewSet`.  
4. How do authentication and permissions work in DRF?  
5. How to handle pagination, throttling, and versioning?  
6. How to make APIs idempotent and retry-safe?  

---

## 9. Caching & Performance — 3 Questions
1. What caching strategies does Django support?  
2. Explain low-level vs per-view caching and Redis integration.  
3. How to invalidate cache and avoid stale data?  

---

## 10. Multi-Tenancy & Architecture — 3 Questions
1. What is multi-tenancy and what are its design approaches?  
2. How to implement multi-tenant architecture in Django (shared schema, db, etc.)?  
3. How to handle migrations and isolation per tenant?  

---

## 11. Testing & Security — 4 Questions
1. How do you write and run tests in Django?  
2. What are `TestCase` and `pytest-django`?  
3. How does Django prevent XSS, CSRF, and SQL injection?  
4. How do you manage secret keys and environment variables securely?  

---

## 12. Async & Task Queues — 3 Questions
1. How does Django support async views and ASGI?  
2. How do Django Channels work for WebSockets?  
3. How to integrate Celery for background tasks?  

---

## 13. Deployment & Scalability — 3 Questions
1. How to deploy Django using Gunicorn + Nginx?  
2. How to scale Django apps (load balancing, caching, workers)?  
3. How to design a scalable, idempotent, and secure API system using Django?  
