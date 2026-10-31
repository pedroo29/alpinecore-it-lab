

\# AlpineCore — Network Plan



\## 1. Topología

\- GL-MT3000: router, gateway y servidor DHCP de AlpineCore.

\- Workstation: conectada al Wi-Fi privado del GL-MT3000.

\- PVE01: conectado por USB-Ethernet al puerto LAN del router.

\- WAN: conexión Repeater variable según la ubicación.



\## 2. Direccionamiento IPv4



| Elemento | Dirección | Configuración |

|---|---|---|

| LAN AlpineCore | 10.77.77.0/24 | Red privada |

| GL-MT3000 | 10.77.77.1 | Gateway y DNS |

| PVE01 | 10.77.77.10/24 | IP estática en vmbr0 |

| Workstation | DHCP | 10.77.77.194 observado |

| DHCP pool | 10.77.77.100–199 | Asignación dinámica |



\## 3. Proxmox

\- Hostname: pve01.alpinecore.test

\- Management interface: vmbr0

\- Physical bridge port: nic0

\- Ethernet MAC: [REDACTED — verified locally]

\- Web UI: https://10.77.77.10:8006



\## 4. Pruebas realizadas

\- Ping workstation → router: PASS

\- Ping workstation → PVE01: PASS

\- Web UI de Proxmox: PASS

\- Acceso local sin Internet: PASS

\- Conectividad externa restaurada: PASS

\- Resolución DNS externa: PASS

\- Acceso web tras reboot: PASS



\## 5. Seguridad y limitaciones

\- No publicar el puerto 8006 hacia Internet.

\- Mantener el Wi-Fi privado protegido.

\- No conectar redes experimentales de VMs a redes ajenas.

\- El FQDN está configurado, pero su resolución DNS local aún no se ha validado.

\- La LAN funciona sin Internet; servicios externos no necesariamente.



