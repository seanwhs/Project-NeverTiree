# 🏗️ **RetireeCorp Website — Technical Architecture Blueprint**

---

## **1. Strategic Objectives & Requirements**

**Business Context (from `Business-Model.md`)**

* **Retirees**: Provide meaningful engagement through advisory, mentorship, and learning.
* **Organizations**: Access verified professionals for advisory or project work.
* **Monetization**: Memberships, transaction fees, enterprise contracts.

**Functional Requirements (from `README.md` + Contribution Guide)**

* **CMS-driven content**: Home, About, Services, Blog, Learning Resources.
* **Portal access for authenticated users**:

  * **Retiree portal**: Manage profile, apply for projects, track courses, mentorship.
  * **Organization portal**: Browse verified retirees, request advisory sessions, manage engagements.
* Modular apps: `users`, `projects`, `lms`, `crm`, `blog`.
* Interactive UI: **HTMX** for partial updates, **Alpine.js** for UI components.
* Responsive design via **Tailwind CSS**.

**Non-Functional Requirements**

* **MySQL backend** (replication-ready for scaling).
* Secure authentication & permissions (Django auth + JWT).
* Progressive enhancement: works even without JavaScript.
* Containerized deployment with horizontal scaling.

---

## **2. Website Scaffold — Modular & Portal-Ready**

```text
retireecorp/
├─ retireecorp/                  # Django project core
│   ├─ settings.py               # MySQL config, INSTALLED_APPS
│   ├─ urls.py                   # Route all apps including portals
│   ├─ wsgi.py / asgi.py
│   └─ templates/
├─ home/                         # Wagtail CMS pages
│   ├─ models.py                 # HomePage, ServicePage, LearningPathPage
│   ├─ wagtail_hooks.py
│   └─ templates/
├─ users/                        # Auth, profiles, portals
│   ├─ models.py                 # Custom User, Roles, Profile
│   ├─ forms.py                  # Registration/Login/Profile
│   ├─ views.py                  # Login, logout, dashboards, portal access
│   └─ urls.py
├─ projects/                     # Project management
│   ├─ models.py                 # Project, Portfolio, Skills, Engagements
│   ├─ views.py                  # Project lists, detail, apply
│   └─ urls.py
├─ lms/ (optional)               # Courses & Learning Pathways
│   ├─ models.py                 # Course, Lesson, Enrollment
│   ├─ views.py
│   └─ urls.py
├─ crm/ (optional)               # Client Relationship Management
│   ├─ models.py                 # Client, Interaction, Notes
│   ├─ views.py
│   └─ urls.py
├─ blog/ (optional)              # Blog content
│   ├─ models.py                 # BlogPage, Categories, Tags
│   ├─ views.py
│   └─ urls.py
├─ templates/                    # Shared templates, partials
├─ static/                       # Tailwind CSS, Alpine.js, HTMX
├─ media/                        # Uploaded media
├─ manage.py
└─ requirements.txt
```

**Key Portal Concepts**

* **Retiree Dashboard**: Projects applied, courses enrolled, mentorship sessions, profile edit.
* **Organization Dashboard**: Search retirees, request advisory, manage projects & engagements.
* Access controlled via **role-based permissions** (`is_retiree`, `is_organization`).
* Partial updates with HTMX for portal sections (projects, courses, interactions).

---

## **3. Backend Architecture (Django 6 + MySQL)**

**Authentication & Authorization**

* Custom User model with **email as username**.
* Roles: Retiree, Organization, Admin.
* **JWT** for API calls; session + token-based auth for web portal.
* Secure dashboard routing: `@login_required` + `@user_passes_test`.

**Business Logic**

* Projects: Applications, skills, engagements.
* LMS: Course progress, enrollments.
* CRM: Interactions, client management.
* Advisory & mentorship workflows.

**Portal-Specific Views Example**

```python
# users/views.py
from django.contrib.auth.decorators import login_required, user_passes_test
from django.shortcuts import render

@login_required
@user_passes_test(lambda u: u.is_retiree)
def retiree_dashboard(request):
    user_projects = request.user.profile.projects.all()
    user_courses = request.user.profile.enrollments.all()
    return render(request, "users/retiree_dashboard.html", {
        "projects": user_projects,
        "courses": user_courses
    })

@login_required
@user_passes_test(lambda u: u.is_organization)
def organization_dashboard(request):
    engagements = request.user.organization.engagements.all()
    return render(request, "users/org_dashboard.html", {"engagements": engagements})
```

---

## **4. Frontend Architecture**

**Server-Rendered HTML**

* Django templates + Wagtail pages.
* **HTMX** for portal interactions without full reloads.
* **Alpine.js** for UI elements: dropdowns, tabs, modals.

**HTMX Portal Example**

```html
<button hx-get="/projects/{{ project.id }}/apply/" hx-target="#project-status">Apply</button>
<div id="project-status"></div>
```

**Alpine.js Example**

```html
<div x-data="{ open: false }">
  <button @click="open = !open">View Details</button>
  <div x-show="open">Project description and instructions</div>
</div>
```

---

## **5. Core Models & Relationships (Portal-Ready)**

| App      | Key Models                              | Relationships                             |
| -------- | --------------------------------------- | ----------------------------------------- |
| users    | User, Profile, Role                     | Profile ↔ Projects, Profile ↔ Enrollments |
| projects | Project, Portfolio, Skill, Engagement   | Skills ↔ Projects (M2M)                   |
| lms      | Course, Lesson, Enrollment              | Course ↔ Lessons ↔ Enrollments            |
| crm      | Client, Interaction, Notes              | Client ↔ Interaction                      |
| blog     | BlogPage, Categories, Tags              | Optional M2M for tags                     |
| home     | HomePage, ServicePage, LearningPathPage | —                                         |

---

## **6. MySQL & Scaling**

* Use `InnoDB` for transactions & foreign keys.
* Master-slave replication for horizontal scaling.
* Index foreign keys and frequently queried fields.
* **Caching & Queues**: Redis/Memcached for sessions & queries, Celery for background tasks.

---

## **7. Deployment & CI/CD**

* **Containers**: Docker for all apps.
* **Horizontal Scaling**: Load balancer + multiple Django/Gunicorn instances.
* **Static & Media**: Nginx + CDN; cloud storage for media (S3, GCS).
* **CI/CD**: GitHub Actions for automated testing, migrations, deployments.
* **Monitoring**: Sentry, uptime monitoring.

---

## **8. Optional Extensions & Modular Apps**

* **LMS**: Courses, Lessons, Enrollments.
* **CRM**: Client management, interactions, engagements.
* **Blog**: Wagtail blog with categories, tags, and search.
* **Analytics**: Engagement metrics across projects, courses, mentorship.

*All apps integrate seamlessly into the portal dashboards.*

---

## **9. Development Workflow**

1. Start with core apps: `home`, `users`, `projects`.
2. Enable portal access with dashboards per role.
3. Add optional apps (`lms`, `crm`, `blog`).
4. Integrate **HTMX** for portal interactions, **Alpine.js** for UI.
5. Tailwind CSS for responsive layouts.
6. Version control: Git with dev/staging/prod branches.
7. Deploy using Docker + MySQL + Nginx/CDN.

---

## **10. Example Portal Integration**

**Retiree Dashboard**

```html
{% extends "base.html" %}
{% block content %}
<h2>Welcome, {{ user.get_full_name }}</h2>

<h3>Your Projects</h3>
<ul>
  {% for project in projects %}
    <li>
      <button hx-get="/projects/{{ project.id }}/details/" hx-target="#project-{{ project.id }}">{{ project.title }}</button>
      <div id="project-{{ project.id }}"></div>
    </li>
  {% endfor %}
</ul>

<h3>Your Courses</h3>
<ul>
  {% for enrollment in courses %}
    <li>{{ enrollment.course.title }} - Progress: {{ enrollment.progress }}%</li>
  {% endfor %}
</ul>
{% endblock %}
```

**Organization Dashboard**

```html
{% extends "base.html" %}
{% block content %}
<h2>Organization Dashboard</h2>
<ul>
  {% for engagement in engagements %}
    <li>{{ engagement.project.title }} - {{ engagement.retiree.get_full_name }}</li>
  {% endfor %}
</ul>
{% endblock %}
```

---

✅ **This polished blueprint now fully integrates**:

* Project-NeverTiree artifacts (`README.md`, `Contribution Guide`, `Business-Model.md`).
* Modular Wagtail/Django scaffold with **portal access**.
* MySQL backend with replication-ready structure.
* Role-based dashboards for Retirees & Organizations.
* **HTMX + Alpine.js** for dynamic interactions.
* Optional LMS, CRM, Blog, and analytics modules.

