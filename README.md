sequenceDiagram
    autonumber
    
    box rgba(173, 216, 230, 0.2) Presentation Layer
        participant App as Mobile App
    end
    box rgba(255, 228, 196, 0.2) API Gateway
        participant GW as API Gateway
    end
    box rgba(144, 238, 144, 0.2) Microservices
        participant Auth as Auth Service
        participant KYC as e-KYC Service
    end
    box rgba(211, 211, 211, 0.2) Data Layer
        participant DB as Relational DB
        participant OSS as Object Storage
    end
    box rgba(255, 192, 203, 0.2) External
        participant Dukcapil as Dukcapil API
    end

    Note over App, Dukcapil: 1. API Registrasi & Upload e-KYC (POST /auth/register)
    App->>GW: POST /auth/register (multipart/form-data)
    GW->>Auth: Route Request
    Auth->>DB: Cek duplikasi Email / KTP
    DB-->>Auth: Result (Tidak ada duplikasi)
    
    Auth->>KYC: Teruskan file foto (KTP & Selfie)
    KYC->>OSS: Simpan file Binary
    OSS-->>KYC: Return URL path gambar
    KYC->>Dukcapil: Hit API Verifikasi Identitas (NIK & Wajah)
    Dukcapil-->>KYC: Response (Match / Valid)
    KYC-->>Auth: Data tervalidasi + Image URL
    
    Auth->>DB: Insert data ke tabel USERS (Hash Password)
    DB-->>Auth: Success
    Auth-->>GW: 201 Created (Return User UUID)
    GW-->>App: HTTP 201 Registration Success

    Note over App, Dukcapil: 2. API Login (POST /auth/login)
    App->>GW: POST /auth/login {email, password}
    GW->>Auth: Route Request
    Auth->>DB: Get User Credentials
    DB-->>Auth: Return Hash Password
    Auth->>Auth: Validate Hash / Biometric Token
    Auth-->>GW: 200 OK (Return JWT Token)
    GW-->>App: HTTP 200 Login Success + Session Token
