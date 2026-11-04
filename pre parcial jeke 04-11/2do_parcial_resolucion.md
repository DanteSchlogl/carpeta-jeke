# 2do. Parcial – Aspectos Legales de la Profesión (resolución para comparar)

Archivo de Packet Tracer 8.2.2: `2do_parcial_maqueta_OSPF_DHCP_DNS_HTTP_PT-8.2.2.pkt`

---

## Preguntas de opción múltiple

| # | Respuesta | Por qué |
|---|---|---|
| 1 | **a** | En vector-distancia el router le cree la distancia al vecino sin conocer la trayectoria. En estado de enlace cada router arma el mapa de la red y calcula el árbol de caminos más cortos (Dijkstra). La b es falsa (solo estado de enlace inunda la topología), la c y la d están invertidas. |
| 2 | **d** | RIPv2 transporta la máscara de subred, por eso admite VLSM y CIDR. Su métrica es el número de saltos (no el ancho de banda), el máximo son 15 saltos (16 = infinito) y es vector-distancia. |
| 3 | **c** | El costo OSPF = ancho de banda de referencia ÷ ancho de banda de la interfaz. |
| 4 | **b** | BGP anuncia la lista completa de sistemas autónomos (AS_PATH) que atraviesa cada prefijo. Si ve su propio AS en la lista, descarta la ruta: detecta bucles sin temporizadores. |
| 5 | **d** | El DISCOVER es un broadcast y no cruza el router. Falta `ip helper-address` en la subinterfaz de la VLAN 30 que apunte al servidor DHCP. |
| VII | **a y d** | Cada vecino anuncia métrica 3, así que R instala métrica 4. Con `maximum-paths 4` instala como máximo 4 de las 5 rutas de igual costo (a). Con los datos no se sabe cuál queda fuera (d). |

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
| R3–R4 | 1,544 Mbps | 100000/1544 = 64,77 → 64 | **64** |
| R2–R4 | 512 kbps | 100000/512 = 195,3 → 195 | **195** |

### b) Dijkstra desde R1

| Paso | Nodo fijado | R2 | R3 | R4 | R5 | R6 |
|---|---|---|---|---|---|---|
| 0 | R1 | 1 (R1) | 10 (R1) | ∞ | ∞ | ∞ |
| 1 | R2 (1) | – | 2 (R2) | 196 (R2) | ∞ | ∞ |
| 2 | R3 (2) | – | – | 66 (R3) | 12 (R3) | ∞ |
| 3 | R5 (12) | – | – | 66 | – | 22 (R5) |
| 4 | R6 (22) | – | – | **23 (R6)** | – | – |
| 5 | R4 (23) | – | – | – | – | – |

| Destino | Mejor trayectoria desde R1 | Costo total |
|---|---|---|
| R2 | R1 → R2 | 1 |
| R3 | R1 → R2 → R3 | 2 |
| R5 | R1 → R2 → R3 → R5 | 12 |
| R6 | R1 → R2 → R3 → R5 → R6 | 22 |
| R4 | R1 → R2 → R3 → R5 → R6 → R4 | **23** |

**Destino R4:** parece que lo mejor es R1→R2→R3→R4 (costo 66), pero pasando por R5 y R6 el costo total es 1+1+10+10+1 = **23**. OSPF suma costos de enlace, no saltos: tres enlaces rápidos y uno lento (R3–R4, costo 64) pierden contra un camino más largo de enlaces baratos. La ruta directa R2–R4 (195) es la peor.

### c) Si cae el enlace R5–R6

- R6 solo se alcanza por R4: R1 → R2 → R3 → R4 → R6 = 2 + 64 + 1 = **67**.
- R4: R1 → R2 → R3 → R4 = **66**.
- R5 no cambia (12).

---

## Problema VIII – VLSM sobre 192.168.10.0/24

### I) ¿Entra el diseño usando /30 en los enlaces?

| Subred | Hosts pedidos | Hosts + gateway | Máscara mínima | Direcciones |
|---|---|---|---|---|
| DHCP+DNS (Red B) | 60 | 61 | /26 (62 útiles) | 64 |
| Mail (Red E) | 50 | 51 | /26 (62 útiles) | 64 |
| HTTP (Red A) | 25 | 26 | /27 (30 útiles) | 32 |
| DB (Red D) | 12 | 13 | /28 (14 útiles) | 16 |
| FTP (Red C) | 6 | 7 | /28 (14 útiles) | 16 |
| 5 enlaces p2p | 2 c/u | – | /30 (2 útiles) | 5 × 4 = 20 |

Total = 64 + 64 + 32 + 16 + 16 + 20 = **212 direcciones ≤ 256**. **Sí entra** y sobran 44 direcciones (192.168.10.212 – .255) para expandir subredes.

Nota: si no se cuenta el gateway, FTP entraría en un /29 (6 útiles) y el total sería 204. Con el gateway del router hacen falta 7 direcciones, por eso la FTP lleva /28.

### II) Cuadro completo

| Red | Req. de hosts | Subred | Bits de red + subred | Máscara | Bits para hosts | Cantidad de hosts | RANGO (primera – última) | Broadcast |
|---|---|---|---|---|---|---|---|---|
| B (DHCP+DNS) | 60 | 192.168.10.0 | 26 | 255.255.255.192 | 6 | 62 | .1 – .62 | .63 |
| E (Mail) | 50 | 192.168.10.64 | 26 | 255.255.255.192 | 6 | 62 | .65 – .126 | .127 |
| A (HTTP) | 25 | 192.168.10.128 | 27 | 255.255.255.224 | 5 | 30 | .129 – .158 | .159 |
| D (DB) | 12 | 192.168.10.160 | 28 | 255.255.255.240 | 4 | 14 | .161 – .174 | .175 |
| C (FTP) | 6 (+1 gw) | 192.168.10.176 | 28 | 255.255.255.240 | 4 | 14 | .177 – .190 | .191 |
| Enlace Norte–Oeste | 2 | 192.168.10.192 | 30 | 255.255.255.252 | 2 | 2 | .193 – .194 | .195 |
| Enlace Norte–Este | 2 | 192.168.10.196 | 30 | 255.255.255.252 | 2 | 2 | .197 – .198 | .199 |
| Enlace Norte–Sur | 2 | 192.168.10.200 | 30 | 255.255.255.252 | 2 | 2 | .201 – .202 | .203 |
| Enlace Oeste–Sur | 2 | 192.168.10.204 | 30 | 255.255.255.252 | 2 | 2 | .205 – .206 | .207 |
| Enlace Este–Sur | 2 | 192.168.10.208 | 30 | 255.255.255.252 | 2 | 2 | .209 – .210 | .211 |
| Libre | – | 192.168.10.212 | – | – | – | 44 | .212 – .255 | – |

(En el rango y el broadcast, el prefijo es 192.168.10.)

### Asignación concreta de IPs

| Dispositivo | Interfaz | IP | Máscara | Gateway |
|---|---|---|---|---|
| Norte | G0/0 (Red A) | 192.168.10.129 | 255.255.255.224 | – |
| Norte | S0/3/0 → Oeste | 192.168.10.193 | 255.255.255.252 | – |
| Norte | S0/3/1 → Este | 192.168.10.197 | 255.255.255.252 | – |
| Norte | S0/2/0 → Sur | 192.168.10.201 | 255.255.255.252 | – |
| Oeste | G0/0 (Red E) | 192.168.10.65 | 255.255.255.192 | – |
| Oeste | S0/3/0 → Norte | 192.168.10.194 | 255.255.255.252 | – |
| Oeste | S0/3/1 → Sur | 192.168.10.205 | 255.255.255.252 | – |
| Este | G0/0 (Red B) | 192.168.10.1 | 255.255.255.192 | – |
| Este | G0/1 (Red C) | 192.168.10.177 | 255.255.255.240 | – |
| Este | S0/3/0 → Norte | 192.168.10.198 | 255.255.255.252 | – |
| Este | S0/3/1 → Sur | 192.168.10.209 | 255.255.255.252 | – |
| Sur | G0/0 (Red D) | 192.168.10.161 | 255.255.255.240 | – |
| Sur | S0/3/0 → Norte | 192.168.10.202 | 255.255.255.252 | – |
| Sur | S0/2/0 → Oeste | 192.168.10.206 | 255.255.255.252 | – |
| Sur | S0/2/1 → Este | 192.168.10.210 | 255.255.255.252 | – |
| HTTP_Server | Fa0 | 192.168.10.130 | 255.255.255.224 | 192.168.10.129 |
| PC0 | Fa0 | 192.168.10.131 | 255.255.255.224 | 192.168.10.129 |
| DHCP_DNS | Fa0 | 192.168.10.2 | 255.255.255.192 | 192.168.10.1 |
| Laptop0, PC3 | Fa0 | DHCP (.6 – .62) | 255.255.255.192 | 192.168.10.1 |
| Mail_Server | Fa0 | 192.168.10.66 | 255.255.255.192 | 192.168.10.65 |
| PC4 | Fa0 | 192.168.10.67 | 255.255.255.192 | 192.168.10.65 |
| DB_Server | Fa0 | 192.168.10.162 | 255.255.255.240 | 192.168.10.161 |
| PC1 | Fa0 | 192.168.10.163 | 255.255.255.240 | 192.168.10.161 |
| FTP_Server | Fa0 | 192.168.10.178 | 255.255.255.240 | 192.168.10.177 |
| PC2 | Fa0 | 192.168.10.179 | 255.255.255.240 | 192.168.10.177 |

DNS de los equipos con IP fija: 192.168.10.2.

---

## Puntos IX y X – lo que hay configurado en la maqueta

- **Topología:** Norte (Red A), Oeste (Red E), Este (Redes B y C) y Sur (Red D), unidos por 5 enlaces seriales: Norte–Oeste, Norte–Este, Norte–Sur, Oeste–Sur, Este–Sur. El extremo DCE lleva `clock rate 64000`: Norte (los tres seriales), Oeste S0/3/1 y Este S0/3/1.
- **OSPF (proceso 1, área 0):** router-id 1.1.1.1 (Norte), 2.2.2.2 (Oeste), 3.3.3.3 (Este), 4.4.4.4 (Sur). Se anuncian las LAN y los enlaces seriales. Las interfaces hacia los usuarios (G0/0 y G0/1 con LAN) son `passive-interface`.
- **Verificación:** `show ip ospf neighbor` en Norte y en Sur muestra las adyacencias en estado **FULL**.
- **DHCP (en DHCP_DNS):** pool de la Red B desde 192.168.10.6 hasta .62, con gateway .1 y DNS .2. Las primeras 5 direcciones (.1 a .5) quedan fuera del rango. Laptop0 recibió .8 y PC3 .7. Ninguna IP estática está dentro del pool.
- **DNS:** registro A `www.alp.com.ar` → 192.168.10.130 (HTTP_Server). `nslookup` desde PC1 lo resuelve.
- **HTTP:** servicio activo en HTTP_Server. Desde PC1 (Red D) y PC4 (Red E) el navegador abre la página usando la URL.
- **Pings verificados:** PC1 → HTTP_Server, PC0 → FTP_Server, PC4 → DB_Server y PC3 → PC4.
- **Cómo repetir las pruebas:** en las PCs, `ping` a un servidor de otra red (el primer ping puede perder un paquete por ARP). En los routers, `show ip ospf neighbor` y `show ip route ospf`.
