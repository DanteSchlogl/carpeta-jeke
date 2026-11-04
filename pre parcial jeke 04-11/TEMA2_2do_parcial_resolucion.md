# TEMA 2 – 2do. Parcial – Aspectos Legales de la Profesión (resolución para comparar)

Archivo de Packet Tracer 8.2.2: `TEMA2_2do_parcial_maqueta_OSPF_DHCP_DNS_HTTP_PT-8.2.2.pkt`

(El Tema 1 está en esta misma carpeta, en los archivos `TEMA1_...`.)

---

## Preguntas de opción múltiple (ojo: en el Tema 2 las opciones están en otro orden)

| # | Respuesta | Por qué |
|---|---|---|
| 3 | **b** | Costo OSPF = ancho de banda de referencia ÷ ancho de banda de la interfaz. |
| 4 | **a** | BGP es de vector de rutas porque anuncia la lista completa de sistemas autónomos (AS_PATH) de cada prefijo. Si se ve su propio AS, descarta la ruta: detecta bucles sin temporizadores. |
| 5 | **b** | El DISCOVER es un broadcast y no cruza el router. Falta `ip helper-address` en la subinterfaz de la VLAN 20 que apunte al servidor DHCP. |
| VII | **b y d** | Ver abajo. |

Las preguntas 1 y 2 de este tema no estaban en las fotos.

### Problema VII (Tema 2)

Cada vecino anuncia la red 10.5.0.0/16 y R le suma 1 salto:

| Vecino | Anuncia | R calcula | Resultado |
|---|---|---|---|
| A | 4 | 5 | instalada |
| B | 4 | 5 | instalada (mismo costo que A) |
| C | 7 | 8 | alcanzable, pero peor que 5: no se instala |
| D | 15 | 16 | 16 = inalcanzable en RIP: no se instala |
| E | 16 | 17 | inalcanzable: no se instala |

- **b) correcta:** instala A y B con métrica 5 y reparte el tráfico entre las dos (balanceo de igual costo; `maximum-paths 4` alcanza para 2).
- **d) correcta:** C es alcanzable pero tiene peor métrica, y D y E quedan en 16 (infinito).
- a) falsa: instala también B. c) falsa: D anuncia 15, pero R le suma 1 y pasa a 16.

---

## Problema VI – OSPF (referencia 100 Mbps, costo = 100 / BW, truncado, mínimo 1)

### a) Costo de los 8 enlaces

| Enlace | Ancho de banda | Cálculo | Costo |
|---|---|---|---|
| R1–R2 | 100 Mbps | 100/100 | **1** |
| R2–R3 | 100 Mbps | 100/100 | **1** |
| R1–R3 | 10 Mbps | 100/10 | **10** |
| R3–R5 | 10 Mbps | 100/10 | **10** |
| R5–R6 | 10 Mbps | 100/10 | **10** |
| R6–R4 | 100 Mbps | 100/100 | **1** |
| R3–R4 | 2,048 Mbps | 100000/2048 = 48,83 → 48 | **48** |
| R2–R4 | 1,544 Mbps | 100000/1544 = 64,77 → 64 | **64** |

### b) Dijkstra desde R1

| Paso | Nodo fijado | R2 | R3 | R4 | R5 | R6 |
|---|---|---|---|---|---|---|
| 0 | R1 | 1 (R1) | 10 (R1) | ∞ | ∞ | ∞ |
| 1 | R2 (1) | – | 2 (R2) | 65 (R2) | ∞ | ∞ |
| 2 | R3 (2) | – | – | 50 (R3) | 12 (R3) | ∞ |
| 3 | R5 (12) | – | – | 50 | – | 22 (R5) |
| 4 | R6 (22) | – | – | **23 (R6)** | – | – |
| 5 | R4 (23) | – | – | – | – | – |

| Destino | Mejor trayectoria desde R1 | Costo total |
|---|---|---|
| R2 | R1 → R2 | 1 |
| R3 | R1 → R2 → R3 | 2 |
| R5 | R1 → R2 → R3 → R5 | 12 |
| R6 | R1 → R2 → R3 → R5 → R6 | 22 |
| R4 | R1 → R2 → R3 → R5 → R6 → R4 | **23** |

**Destino R4:** parece que lo mejor es R1→R2→R3→R4 (costo 50), pero pasando por R5 y R6 el costo total es 1+1+10+10+1 = **23**. OSPF suma costos de enlace, no saltos: un camino más largo con enlaces baratos (los de 100 Mbps cuestan 1) gana a uno corto con un enlace caro (R3–R4 cuesta 48). Los costos de R2–R4 (64) y R1–R3 (10) no sirven para llegar más rápido.

### c) Si cae el enlace R5–R6

- R4: R1 → R2 → R3 → R4 = 1 + 1 + 48 = **50** (la otra opción, por R2–R4, cuesta 65).
- R6: ya no se llega por R5; solo por R4: R1 → R2 → R3 → R4 → R6 = 50 + 1 = **51**.
- R5 no cambia (12).

---

## Problema VIII – VLSM sobre 172.20.0.0/16 (banco)

### I) ¿Entra el diseño usando /30 en los enlaces?

| Subred | Hosts pedidos | Máscara mínima | Por qué | Direcciones |
|---|---|---|---|---|
| DHCP+DNS (Red B) | 2047 | /20 (4094 útiles) | un /21 solo da 2046: falta 1 | 4096 |
| HTTP (Red A) | 511 | /22 (1022 útiles) | un /23 solo da 510: falta 1 | 1024 |
| Mail (Red E) | 510 | /23 (510 útiles) | justo | 512 |
| FTP (Red C) | 15 | /27 (30 útiles) | un /28 solo da 14: falta 1 | 32 |
| DB (Red D) | 14 | /28 (14 útiles) | justo | 16 |
| 5 enlaces p2p | 2 c/u | /30 (2 útiles) | | 5 × 4 = 20 |

Total = 4096 + 1024 + 512 + 32 + 16 + 20 = **5700 direcciones ≤ 65.536**. **Sí entra** y sobran 59.836, así que hay lugar de sobra para crecer en subredes.

Nota: Mail (510) y DB (14) entran justo, sin lugar para el gateway del router. Si el profe pide contarlo, serían Mail /22 y DB /27. El enunciado dice "capacidad para N hosts", y esta resolución usa la capacidad exacta.

### II) Cuadro completo

| Red | Req. de hosts | Subred | Bits de red + subred | Máscara | Bits para hosts | Cantidad de hosts | RANGO (primera – última) | Broadcast |
|---|---|---|---|---|---|---|---|---|
| B (DHCP+DNS) | 2047 | 172.20.0.0 | 20 | 255.255.240.0 | 12 | 4094 | 172.20.0.1 – 172.20.15.254 | 172.20.15.255 |
| A (HTTP) | 511 | 172.20.16.0 | 22 | 255.255.252.0 | 10 | 1022 | 172.20.16.1 – 172.20.19.254 | 172.20.19.255 |
| E (Mail) | 510 | 172.20.20.0 | 23 | 255.255.254.0 | 9 | 510 | 172.20.20.1 – 172.20.21.254 | 172.20.21.255 |
| C (FTP) | 15 | 172.20.22.0 | 27 | 255.255.255.224 | 5 | 30 | 172.20.22.1 – 172.20.22.30 | 172.20.22.31 |
| D (DB) | 14 | 172.20.22.32 | 28 | 255.255.255.240 | 4 | 14 | 172.20.22.33 – 172.20.22.46 | 172.20.22.47 |
| Enlace Norte–Oeste | 2 | 172.20.22.48 | 30 | 255.255.255.252 | 2 | 2 | 172.20.22.49 – 172.20.22.50 | 172.20.22.51 |
| Enlace Norte–Este | 2 | 172.20.22.52 | 30 | 255.255.255.252 | 2 | 2 | 172.20.22.53 – 172.20.22.54 | 172.20.22.55 |
| Enlace Norte–Sur | 2 | 172.20.22.56 | 30 | 255.255.255.252 | 2 | 2 | 172.20.22.57 – 172.20.22.58 | 172.20.22.59 |
| Enlace Oeste–Sur | 2 | 172.20.22.60 | 30 | 255.255.255.252 | 2 | 2 | 172.20.22.61 – 172.20.22.62 | 172.20.22.63 |
| Enlace Este–Sur | 2 | 172.20.22.64 | 30 | 255.255.255.252 | 2 | 2 | 172.20.22.65 – 172.20.22.66 | 172.20.22.67 |
| Libre | – | 172.20.22.68 | – | – | – | – | 172.20.22.68 – 172.20.255.255 | – |

### Asignación concreta de IPs

| Dispositivo | Interfaz | IP | Máscara | Gateway |
|---|---|---|---|---|
| Norte | G0/0 (Red A) | 172.20.16.1 | 255.255.252.0 | – |
| Norte | S0/3/0 → Oeste | 172.20.22.49 | 255.255.255.252 | – |
| Norte | S0/3/1 → Este | 172.20.22.53 | 255.255.255.252 | – |
| Norte | S0/2/0 → Sur | 172.20.22.57 | 255.255.255.252 | – |
| Oeste | G0/0 (Red E) | 172.20.20.1 | 255.255.254.0 | – |
| Oeste | S0/3/0 → Norte | 172.20.22.50 | 255.255.255.252 | – |
| Oeste | S0/3/1 → Sur | 172.20.22.61 | 255.255.255.252 | – |
| Este | G0/0 (Red B) | 172.20.0.1 | 255.255.240.0 | – |
| Este | G0/1 (Red C) | 172.20.22.1 | 255.255.255.224 | – |
| Este | S0/3/0 → Norte | 172.20.22.54 | 255.255.255.252 | – |
| Este | S0/3/1 → Sur | 172.20.22.65 | 255.255.255.252 | – |
| Sur | G0/0 (Red D) | 172.20.22.33 | 255.255.255.240 | – |
| Sur | S0/3/0 → Norte | 172.20.22.58 | 255.255.255.252 | – |
| Sur | S0/2/0 → Oeste | 172.20.22.62 | 255.255.255.252 | – |
| Sur | S0/2/1 → Este | 172.20.22.66 | 255.255.255.252 | – |
| HTTP_Server | Fa0 | 172.20.16.2 | 255.255.252.0 | 172.20.16.1 |
| PC0 | Fa0 | 172.20.16.3 | 255.255.252.0 | 172.20.16.1 |
| DHCP_DNS | Fa0 | 172.20.0.2 | 255.255.240.0 | 172.20.0.1 |
| Laptop0, PC3 | Fa0 | DHCP (172.20.0.6 – 172.20.15.254) | 255.255.240.0 | 172.20.0.1 |
| Mail_Server | Fa0 | 172.20.20.2 | 255.255.254.0 | 172.20.20.1 |
| PC4 | Fa0 | 172.20.20.3 | 255.255.254.0 | 172.20.20.1 |
| FTP_Server | Fa0 | 172.20.22.2 | 255.255.255.224 | 172.20.22.1 |
| PC2 | Fa0 | 172.20.22.3 | 255.255.255.224 | 172.20.22.1 |
| DB_Server | Fa0 | 172.20.22.34 | 255.255.255.240 | 172.20.22.33 |
| PC1 | Fa0 | 172.20.22.35 | 255.255.255.240 | 172.20.22.33 |

DNS de los equipos con IP fija: 172.20.0.2.

---

## Puntos IX y X – lo que hay configurado en la maqueta

- **Topología:** igual que en el Tema 1: Norte (Red A), Oeste (Red E), Este (Redes B y C) y Sur (Red D), unidos por 5 enlaces seriales. El extremo DCE lleva `clock rate 64000`: los tres seriales de Norte, Oeste S0/3/1 y Este S0/3/1.
- **OSPF (proceso 1, área 0):** router-id 1.1.1.1 (Norte), 2.2.2.2 (Oeste), 3.3.3.3 (Este), 4.4.4.4 (Sur). Se anuncian las LAN con wildcard (por ejemplo 172.20.16.0 0.0.3.255 para el /22) y los enlaces seriales. Las interfaces hacia los usuarios son `passive-interface`.
- **Verificación:** `show ip ospf neighbor` en Norte (3 vecinos) y en Este (2 vecinos) muestra todo en **FULL**.
- **DHCP (en DHCP_DNS):** pool de la Red B desde 172.20.0.6 hasta 172.20.15.254, con gateway 172.20.0.1 y DNS 172.20.0.2. Las primeras 5 direcciones (172.20.0.1 a .5) quedan fuera del rango. Laptop0 recibió 172.20.0.8 y PC3 172.20.0.7. Ninguna IP estática está dentro del pool.
- **DNS:** registro A `www.alp.com.ar` → 172.20.16.2 (HTTP_Server). `nslookup` desde PC1 lo resuelve.
- **HTTP:** desde el navegador de PC1 (Red D), `http://www.alp.com.ar` abre la página.
- **Pings verificados:** PC1 → HTTP_Server, PC4 → DB_Server, PC0 → FTP_Server y PC3 → PC2 (172.20.22.3).
- **Truco de Packet Tracer:** si cambias el DNS de una PC y `ipconfig /all` sigue mostrando el viejo, pásala a DHCP y vuelve a IP fija con el DNS nuevo.
