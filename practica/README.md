# Ejercicios de práctica (Packet Tracer 8.2.2)

Todos son redes con direccionamiento estático. Cada archivo se probó con un ping entre PCs de distintas LAN (alguna respuesta inicial puede perderse por ARP/STP; basta repetir el ping).

| # | Archivo | Qué practica | Prueba |
|---|---------|--------------|--------|
| 1 | `01_LAN_unica_PT-8.2.2.pkt` | Un router, un switch y 3 PCs en 192.168.0.0/24 | A_PC1A → 192.168.0.4 |
| 2 | `02_dos_routers_rutas_estaticas_PT-8.2.2.pkt` | Dos routers unidos por 10.0.0.0/30 con rutas estáticas; LAN 192.168.0.0/24 y 192.168.1.0/24 | B_PC1A → 192.168.1.3 |
| 3 | `03_tres_routers_OSPF_PT-8.2.2.pkt` | Triángulo de 3 routers con OSPF (área 0); LAN 192.168.0/1/2.0/24 | C_PC1A → 192.168.2.2 |
| 4 | `04_tres_routers_en_linea_estaticas_PT-8.2.2.pkt` | 3 routers en línea con rutas estáticas; LAN 192.168.10/20/30.0/24 | D_PC1A → 192.168.30.3 |
| 5 | `05_router_con_tres_LANs_PT-8.2.2.pkt` | Un router con 3 LAN (192.168.1/2/3.0/24), 2 PCs por LAN | E_PC1A → 192.168.3.3 |

Ideas para variar: cambiar las IPs, agregar un servidor, pasar el ejercicio 4 a RIP o a OSPF, o romper una ruta a propósito y encontrar el error.
