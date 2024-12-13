# Exercise Requirements

## IaaS VM Deployment and Configuration

- Only IaaS Virtual Machines from **Azure**, **Oracle Cloud**, or **DigitalOcean** can be used.
- The environment used for the deployment must be clean (no pre-existing resources).
- The operating system must be **Ubuntu 22.04 LTS**.
- Both internal and external firewalls must be active and properly configured.

## Objectives

### Part 1 (5 Points)
- Accessing the URL `vhost.<domain>.com` should display:

```
Has accedido a la página del examen
Soy Alex.
```
  

- Accessing the machine via its IP should display:

```
No se puede acceder por IP
```

### Part 2 (2 Points)
- Enable **HTTPS** access for the domain configured in Part 1.

### Part 3 (1 Point)
- Access the website directly via HTTPS without receiving a browser warning about the certificate being untrusted (use a valid certificate).

### Part 4 (3 Points)
- Accessing the `/redireccion` resource should redirect users to the institute's website using `.htaccess`. For example:
  
  - `/redireccion` should redirect to `http://www.ies-azarquiel.es`. 

---

## Plan and Execution
### Tools and Environment
- **IaaS Provider**: Azure.
- **Operating System**: Ubuntu 22.04 LTS.
- **Web Server**: Apache2.
- **Firewall**: Configured using UFW and cloud provider's firewall settings.



---
