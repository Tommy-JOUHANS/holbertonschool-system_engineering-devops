#web_infrastructure_design

---
##TASK0 :Simple web stack

**0_simple_web_stack**

flowchart TD
    User["👤 User\n(browser)"]
    DNS["🌐 DNS Server\nfoobar.com → 8.8.8.8"]
    
    subgraph Server["🖥️ Server — IP: 8.8.8.8"]
        Nginx["⚙️ Web Server\n(Nginx)"]
        AppServer["🚀 Application Server\n(PHP-FPM / Gunicorn)"]
        CodeBase["📁 Application Files\n(codebase)"]
        MySQL["🗄️ Database\n(MySQL)"]
    end

    User -->|"1. DNS query: what is www.foobar.com?"| DNS
    DNS -->|"2. A record → 8.8.8.8"| User
    User -->|"3. HTTP/HTTPS request"| Nginx
    Nginx -->|"4. forwards dynamic request"| AppServer
    AppServer -->|"5. reads codebase"| CodeBase
    AppServer -->|"6. SQL query"| MySQL
    MySQL -->|"7. data"| AppServer
    AppServer -->|"8. HTML response"| Nginx
    Nginx -->|"9. HTTP response"| User


---
    ##TASK 1 : Distributed web infrastructure
    **1-distributed_web_infrastructure**

flowchart TD
    User["👤 User\n(browser)"]
    DNS["🌐 DNS Server\nwww.foobar.com → 8.8.8.8"]
    LB["⚖️ Load Balancer\n(HAProxy — Round Robin)\n⚠️ SPOF"]

    subgraph Server1["🖥️ Server 1"]
        Nginx1["⚙️ Nginx"]
        App1["🚀 App Server"]
        Code1["📁 Codebase"]
    end

    subgraph Server2["🖥️ Server 2"]
        Nginx2["⚙️ Nginx"]
        App2["🚀 App Server"]
        Code2["📁 Codebase"]
    end

    subgraph DB_Cluster["🗄️ Database Cluster"]
        Primary["Primary (Master)\n✏️ READ + WRITE"]
        Replica["Replica (Slave)\n📖 READ only"]
        Primary -->|"replication"| Replica
    end

    User -->|"DNS query"| DNS
    DNS -->|"8.8.8.8"| User
    User -->|"HTTP request"| LB
    LB -->|"request 1, 3, 5..."| Nginx1
    LB -->|"request 2, 4, 6..."| Nginx2
    Nginx1 --> App1 --> Code1
    Nginx2 --> App2 --> Code2
    App1 -->|"writes"| Primary
    App2 -->|"writes"| Primary
    App1 -->|"reads"| Replica
    App2 -->|"reads"| Replica