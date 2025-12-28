# Nginx HTTP Methods Guide

A beginner-friendly guide to HTTP methods and how to handle them in Nginx, including common errors like 405 and how to manage static files, forms, and APIs effectively.

---

## 1️⃣ What are HTTP Methods?

Every request to a server specifies **what the client or application wants to do** using an HTTP Method.

### Common Methods:

| Method      | Simple Explanation    | Use Case                                      |
| ----------- | --------------------- | --------------------------------------------- |
| **GET**     | Read data             | Viewing web pages, fetching files             |
| **POST**    | Send data             | Forms, file uploads                           |
| **PUT**     | Replace existing data | Updating a database record                    |
| **PATCH**   | Modify part of data   | Changing a single field in a record           |
| **DELETE**  | Delete data           | Removing a record or file                     |
| **HEAD**    | Fetch headers only    | Check if a file exists without downloading it |
| **OPTIONS** | Check allowed methods | Used in APIs and CORS                         |

💡 Summary: Methods tell the server what action the client wants to perform.

---

## 2️⃣ GET and POST on Static Files

**Static files:** Images, CSS, JS, HTML (files that are served as-is, without processing)

| Method | Behavior on static files |
| ------ | ------------------------ |
| GET    | File is returned ✅       |
| POST   | 405 Method Not Allowed ❌ |

---

## 3️⃣ What is a 405 Error?

* When a client sends a method that the server does not allow on that resource:

```
405 Method Not Allowed
```

Example: Sending POST to `/image.jpg` → 405 error.

---

## 4️⃣ Using `error_page 405 =200 $uri` in Nginx

```nginx
error_page 405 =200 $uri;
```

### Simple Explanation:

* Instead of returning 405
* The server responds with **200 OK** and displays the original file

### Why Useful:

1. Prevents errors on static files
2. Prevents errors on forms or SPAs (Single Page Applications)

### Practical Example:

```nginx
location /images/ {
    root /var/www/html;
    error_page 405 =200 $uri;
}
```

* GET → Image is displayed
* POST → 405 is converted to 200 → same image displayed

---

## 5️⃣ How Nginx Handles Methods

* Nginx checks the path `/` or `/static`
* GET → file is returned
* POST → default 405 → can override with `error_page` to return 200 instead

---

## 6️⃣ Summary (Simplified)

1. **Methods** = instructions from the client to the server
2. **405** = Method not allowed
3. **error_page 405 =200 $uri** = allow showing file even if the method is incorrect
4. **Use case** = prevent errors on static files, forms, or SPAs

---

## 7️⃣ Complete HTTP Methods Table with Nginx Default Behavior & Overrides

| Method  | Default Nginx Behavior      | How to Override / Handle                                     |
| ------- | --------------------------- | ------------------------------------------------------------ |
| GET     | Returns the file or content | Usually no change needed                                     |
| POST    | 405 on static files         | `error_page 405 =200 $uri;` or route to backend              |
| PUT     | 405                         | `error_page 405 =200 $uri;` or route to backend              |
| PATCH   | 405                         | route to backend                                             |
| DELETE  | 405                         | route to backend                                             |
| HEAD    | Returns headers only        | Default enabled, no change needed                            |
| OPTIONS | 200 or 405                  | `add_header Allow "GET, POST, OPTIONS";` to explicitly allow |

* This table shows exactly **how each HTTP method behaves by default in Nginx** and how you can override the behavior using `error_page` or other directives.

---

## 8️⃣ Routing Requests to Backend

Sometimes, Nginx cannot process certain methods (like POST, PUT, PATCH, DELETE) on static files. In these cases, we **route the request to a backend server** that can handle it.

### What "Route to Backend" Means:

* Nginx acts as a **reverse proxy** and forwards the request to another server (backend) for processing.
* The backend can be an application server, API server, or any service that performs the required action.

### Why Use It:

1. Allows processing of POST/PUT/PATCH/DELETE requests even on static or proxy paths.
2. Keeps Nginx lightweight, handling only reverse proxying and static content.
3. Essential for API endpoints and dynamic content.

### Simple Example:

```nginx
location /api/ {
    proxy_pass http://192.168.0.140:80;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
}
```

**Explanation:**

* Any request to `/api/` is sent to the backend server at `192.168.0.140:80`.
* The backend can handle POST, PUT, PATCH, DELETE, or other methods.
* Nginx only forwards the request and headers, making it easier to scale and manage.

### When to Use:

* API endpoints for web applications.
* Dynamic processing of files or data.
* When Nginx alone cannot process certain request types on static paths.

---

This guide can be placed directly in GitHub and is designed to be beginner-friendly, practical, and scientifically accurate.

