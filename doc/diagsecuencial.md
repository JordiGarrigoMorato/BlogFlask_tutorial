## Diagrama 

```mermaid

sequenceDiagram
    actor U as Usuari
    participant N as Navegador
    participant F as Flask (app)
    participant A as auth (Blueprint)
    participant B as blog (Blueprint)
    participant D as db.py (get_db)
    participant S as SQLite (flaskr.sqlite)

    Note over U,S: Abans de CADA petició (before_app_request)
    N->>F: Qualsevol petició (amb cookie de sessió)
    F->>A: load_logged_in_user()
    alt Hi ha user_id a la sessió
        A->>D: get_db()
        D-->>A: connexió (g.db)
        A->>S: SELECT * FROM user WHERE id = ?
        S-->>A: fila de l'usuari
        A-->>F: g.user = usuari
    else No hi ha sessió
        A-->>F: g.user = None
    end

    Note over U,S: Login (POST /auth/login)
    U->>N: Omple username i password
    N->>A: POST /auth/login
    A->>D: get_db()
    A->>S: SELECT * FROM user WHERE username = ?
    S-->>A: usuari o None
    alt Usuari inexistent
        A-->>N: flash("Incorrect username.") + login.html
    else Contrasenya incorrecta (check_password_hash)
        A-->>N: flash("Incorrect password.") + login.html
    else Credencials correctes
        A->>A: session.clear() i session["user_id"] = id
        A-->>N: 302 redirect a index + cookie de sessió
    end

    Note over U,S: Crear post (POST /create)
    U->>N: Omple títol i cos
    N->>B: POST /create
    B->>B: login_required: g.user és None?
    alt Anònim
        B-->>N: 302 redirect a /auth/login
    else Autenticat
        alt Títol buit
            B-->>N: flash("Title is required.") + create.html
        else Títol vàlid
            B->>D: get_db()
            B->>S: INSERT INTO post (title, body, author_id)
            B->>S: commit()
            B-->>N: 302 redirect a blog.index

    ```