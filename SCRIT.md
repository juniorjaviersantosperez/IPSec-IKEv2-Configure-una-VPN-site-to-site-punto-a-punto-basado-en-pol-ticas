# IPSec Site-to-Site con IKEv2 (Crypto Map)

## Topología

### R2
- WAN: `200.1.15.2/24`
- LAN: `10.15.99.1/24`

### R3
- WAN: `200.1.99.2/24`
- LAN: `192.168.99.1/24`

### Credenciales VPN
- Pre-Shared Key: `1599vpn`
- Cifrado: `AES-256`
- Integridad: `SHA256`
- Grupo DH: `14`

---

# Configuración de R2

```bash
enable
configure terminal

crypto ikev2 proposal PROP-1599
 encryption aes-cbc-256
 integrity sha256
 group 14
exit

crypto ikev2 policy POL-1599
 proposal PROP-1599
exit

crypto ikev2 keyring KR-1599
 peer R3
  address 200.1.99.2
  pre-shared-key 1599vpn
 exit
exit

crypto ikev2 profile PROF-1599
 match identity remote address 200.1.99.2 255.255.255.255
 authentication remote pre-share
 authentication local pre-share
 keyring local KR-1599
exit

crypto ipsec transform-set TS-1599v2 esp-aes 256 esp-sha256-hmac
 mode tunnel
exit

crypto ipsec profile IPSEC-PROF-1599
 set transform-set TS-1599v2
 set ikev2-profile PROF-1599
exit

ip access-list extended ACL-VPN-1599v2
 permit ip 10.15.99.0 0.0.0.255 192.168.99.0 0.0.0.255
exit

crypto map CMAP-1599v2 10 ipsec-isakmp
 set peer 200.1.99.2
 set transform-set TS-1599v2
 set ikev2-profile PROF-1599
 match address ACL-VPN-1599v2
exit

interface ethernet 0/0
 crypto map CMAP-1599v2
exit

end
write memory
```

---

# Configuración de R3

```bash
enable
configure terminal

crypto ikev2 proposal PROP-1599
 encryption aes-cbc-256
 integrity sha256
 group 14
exit

crypto ikev2 policy POL-1599
 proposal PROP-1599
exit

crypto ikev2 keyring KR-1599
 peer R2
  address 200.1.15.2
  pre-shared-key 1599vpn
 exit
exit

crypto ikev2 profile PROF-1599
 match identity remote address 200.1.15.2 255.255.255.255
 authentication remote pre-share
 authentication local pre-share
 keyring local KR-1599
exit

crypto ipsec transform-set TS-1599v2 esp-aes 256 esp-sha256-hmac
 mode tunnel
exit

crypto ipsec profile IPSEC-PROF-1599
 set transform-set TS-1599v2
 set ikev2-profile PROF-1599
exit

ip access-list extended ACL-VPN-1599v2
 permit ip 192.168.99.0 0.0.0.255 10.15.99.0 0.0.0.255
exit

crypto map CMAP-1599v2 10 ipsec-isakmp
 set peer 200.1.15.2
 set transform-set TS-1599v2
 set ikev2-profile PROF-1599
 match address ACL-VPN-1599v2
exit

interface ethernet 0/0
 crypto map CMAP-1599v2
exit

end
write memory
```

---

# Verificación

## Estado de IKEv2

```bash
show crypto ikev2 sa
```

## Estado de la sesión VPN

```bash
show crypto session
```

## Estadísticas IPSec

```bash
show crypto ipsec sa
```

## Tabla de rutas

```bash
show ip route
```

## Prueba de conectividad VPN

```bash
ping 10.15.99.1 source 192.168.99.1
```

---

# Resultado Esperado

```text
IKEv2 SA: READY
IPSec SA: ACTIVE
Paquetes encapsulados y decapsulados incrementando
Ping exitoso entre 10.15.99.0/24 y 192.168.99.0/24
```
