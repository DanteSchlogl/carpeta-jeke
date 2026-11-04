# Apuntes pre parcial – Aspectos Legales de la Profesión (04/11)

Revisión de las preguntas y respuestas marcadas del 1er parcial. ✅ = correcta, ⚠️ = correcta con matiz o error a corregir.

## Direccionamiento IP

| Pregunta | Respuesta | Estado y explicación |
|---|---|---|
| 127.58.36.255 | b) Loopback | ✅ Todo 127.0.0.0/8 es loopback. |
| 65.534 hosts | b) 129.10.0.0 | ✅ Clase B (/16): 2¹⁶ − 2 = 65.534. 226.x es multicast; 192.x es clase C (254 hosts). |
| PC en 192.168.5.33/27 | d) 192.168.5.62 | ✅ Red 192.168.5.32/27, hosts .33–.62, broadcast .63. .32 es la red; .12 y .30 son de la red .0/27; .72 es de otra red. |
| PC en 192.168.16.0/23 | a, b, c, d, e | ✅ Hosts 192.168.16.1 – 192.168.17.254; ninguna es red (16.0) ni broadcast (17.255). |
| Broadcast de subredes /20 | b, e, f | ✅ Subredes de a 16 en el 3.er octeto. 172.16.32.0/20 → **47.255**; 172.16.64.0/20 → **79.255**; el switch 172.16.82.90/20 está en 172.16.80.0/20 → **95.255**. |

## Routing

| Pregunta | Respuesta | Estado y explicación |
|---|---|---|
| RIP | c) Vector-distancia, convergencia lenta, hold-down, split horizon, poison reverse | ✅ Se escribe *poison*, no "Poisson". |
| Hold Down de RIP | Ignora la información de inaccesibilidad por un período (60 s) | ✅ Tras recibir que una red es inalcanzable, el router ignora anuncios sobre esa red durante 60 s, para evitar que rutas viejas la "revivan" (cuenta hasta el infinito). Se complementa con split horizon y poison reverse. |
| OSPF | c) Define áreas y balance de carga | ✅ |
| Routers de distintos AS | Ruteadores de borde | ✅ |
| GGP | Vector-distancia, formato fijo, solo núcleos, distancias en saltos | ✅ Fue reemplazado por EGP y después BGP. |
| Similitud LS vs VD | a) Calcula la distancia a los adyacentes | ✅ Solo la a). Dijkstra (b) y envío a todos (c) son del estado de enlace. |
| Router NO núcleo sin ruta | a) Ruta de omisión (default) | ✅ |
| Router de núcleo sin ruta | e) Ninguna | ✅ Descarta el datagrama y avisa al origen con ICMP "destino inalcanzable". |
| Capa de red | b) Elección de trayectorias | ✅ Corrección de errores y control de flujo son de enlace y transporte. |

### Tabla del router K (actualización con lo que anuncia J)

Se suma 1 a la distancia de J y se compara. Si el siguiente salto actual ya es J, se acepta su valor aunque empeore.

| Destino | Distancia | Ruta | Motivo |
|---|---|---|---|
| Red 1 | 0 | directa | |
| Red 2 | 0 | directa | |
| Red 4 | 4 | por J | 3+1 < 8 |
| Red 17 | 5 | por M | J ofrece 7 |
| Red 21 | 5 | por J | nueva, 4+1 |
| Red 24 | 6 | por J | 5+1 |
| Red 30 | 2 | por Q | J ofrece 11 |
| Red 42 | 4 | por J | siguiente salto es J: 3+1 |

## IP: datagrama y fragmentación

**¿Cambia el datagrama IP al pasar de una Ethernet a otra igual?** Sí: baja el TTL en 1 y se recalcula el checksum. Las IP no cambian (no hay NAT) y no se fragmenta (mismo MTU). Cambian las MAC, pero eso es de la trama.

### Consigna 14 (fragmentación)

Datos 1472 B + TCP 20 B = 1492 B de payload IP; con cabecera IP, 1512 B. ID = 1000, DF = 0.

- **A→Oeste:** MTU 1526 − 14 (Ethernet) = 1512. Entra justo: 1 datagrama, Total Length 1512, **MF = 0**, offset 0. (La solución de la hoja decía MF = 1: está mal.)
- **Oeste→Centro:** MTU 520 − 20 = 500 → payload máximo 480 (múltiplo de 8). 1492 = 480×3 + 52.

| Total Length | MF | Offset |
|---|---|---|
| 500 | 1 | 0 |
| 500 | 1 | 60 |
| 500 | 1 | 120 |
| 72 | 0 | 180 |

- **Centro→Sur:** MTU 300 − 12 = 288 → payload máximo 268, pero debe ser múltiplo de 8 → **264** (33 bloques). Cada fragmento de 480 se parte en 264 + 216; el de 52 pasa sin partir.

| Total Length | MF | Offset |
|---|---|---|
| 284 | 1 | 0 |
| 236 | 1 | 33 |
| 284 | 1 | 60 |
| 236 | 1 | 93 |
| 284 | 1 | 120 |
| 236 | 1 | 153 |
| 72 | 0 | 180 |

**a)** Total Length máximo por enlace: 1512, 500 y 288 (el mayor realmente enviado en el último es 284). **c)** Llegan **7 fragmentos** al destino. Suman 1492 B (3×480 + 52).

## TCP y servicios

| Pregunta | Respuesta | Estado y explicación |
|---|---|---|
| Ventana 65.535 B, 1 Gbps, 10 ms | 26,2 Mbps | ⚠️ RTT = 20 ms → 50 ventanas/s → 50 × 65.535 × 8 = 26,214 Mbps. Eficiencia **2,56%** (÷1024) o **2,62%** (÷1000); no 2,55%. |
| Algoritmo de Karn | c) Ambigüedad del acuse de recibo | ✅ No se mide el RTT de segmentos retransmitidos. Lo que se multiplica por una constante en cada retransmisión es el **RTO** (no "RRT"). |
| Ventana tonta, receptor | c) | ⚠️ Es la que corresponde entre las opciones (la a es del emisor, Nagle). El concepto exacto (Clark): no anunciar ventanas pequeñas hasta tener al menos min(MSS, mitad del buffer). |
| TFTP | e) A y B | ✅ Formato especificado y varios clientes en un servidor. No tiene autenticación. |
| DHCP | a, b, c | ✅ Asignación manual (preasignada), automática (recuerda la anterior) y dinámica por arrendamiento. |

## Packet Tracer

**Serial Sydney (DCE), conectado a Melbourne 201.100.53.1/24:** correctas a, b y d.

```
Sydney(config-if)# ip address 201.100.53.2 255.255.255.0
Sydney(config-if)# clock rate 56000
Sydney(config-if)# no shutdown
```

La c) usa la IP de Melbourne y una máscara /27; la e) (`ip host`) no configura el enlace. El `clock rate` va solo en el extremo DCE.

### Consigna 15 (maqueta Oeste / Centro / Sur)

Enunciado: red tipo C con máscaras ≥ /25 y VLSM; ruteo estático para las redes A y C a través de Oeste, Centro y Sur; RIP para el resto; ping en ambos sentidos.

- Routers 2620XM con enlaces seriales /30 (en la hoja: .65/.66, .69/.70, .73/.74).
- Hosts escritos a mano: A ≈ 200, B ≈ 500, C = 1500, D = 50.
- Con 1500 hosts una red clase C /25 o más no alcanza: el ejercicio tiene una inconsistencia para consultarle al profe.
- En la hoja hay una nota en verde: "Desfasaje c/gráfico y maqueta" (la maqueta entregada no coincide con el gráfico).
- Práctica relacionada: `practica/06_tres_routers_RIP_VLSM_PT-8.2.2.pkt`.
