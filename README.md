# OWASP Top 10:2021
## A02:2021 – Fallas Criptográficas :unlock:

###### tags: `Tag(OWASPTop10,PPS,MongoDB,Cryptographic,Failures,Snake)`

## :warning: Vulnerabilidad

### Contexto

La exposición de datos sensibles ha subido al número 2 en la lista, siendo más un síntoma amplio que una causa raíz. Esto se debe a fallos relacionados con la criptografía o su ausencia, lo que a menudo conduce a la exposición de datos. Algunas vulnerabilidades incluidas son el uso de contraseñas en código fuente, algoritmos criptográficos débiles y falta de entropía.

| CWEs mapeadas | Tasa de incidencia máx | Tasa de incidencia prom | Explotabilidad ponderada prom | Impacto ponderado prom | Cobertura máx | Cobertura prom | Incidencias totales | Total CVEs |
|---------------|------------------------|-------------------------|-------------------------------|------------------------|---------------|----------------|---------------------|------------|
| 29            | 46.44%                 | 4.49%                   | 7.29                          | 6.81                   | 79.33%        | 34.85%         | 233,788             | 3,075      |


### Descripción General

`A02:2021 – Fallas Criptográficas` se refiere a una categoría específica de vulnerabilidades en el contexto del proyecto OWASP Top 10.

Muchas aplicaciones web y API `no protegen adecuadamente los datos confidenciales`, como los financieros, sanitarios y la PII. Los atacantes pueden robar o modificar dichos datos débilmente protegidos para realizar fraudes con tarjetas de crédito, robo de identidad u otros delitos. Los datos confidenciales pueden verse comprometidos sin protección adicional, como cifrado en reposo o en tránsito, y requieren precauciones especiales cuando se intercambian con el navegador.

- La protección de datos sensibles es fundamental, especialmente cuando están sujetos a regulaciones como GDPR o PCI DSS. Algunos aspectos importantes a considerar incluyen:
- Transmisión segura de datos: ¿Los datos se transmiten en texto claro? Es necesario verificar tanto el tráfico externo como el interno.
- Uso de criptografía: ¿Se utilizan algoritmos débiles o desactualizados? ¿Las claves criptográficas se gestionan de forma adecuada? ¿Se incluyen en repositorios de código fuente?
- Forzado de cifrado: ¿Se aplican directivas de seguridad adecuadas, como los encabezados HTTP?
- Validación de certificados: ¿Se valida adecuadamente el certificado de servidor y la cadena de confianza?
- Seguridad en la generación de claves: ¿Se generan vectores de inicialización de manera segura? ¿Se utilizan generadores de aleatoriedad apropiados?
- Funciones hash y métodos de relleno: ¿Se utilizan funciones hash obsoletas? ¿Se aplican métodos de relleno inseguros?
- Vulnerabilidades de mensajes de errores criptográficos: ¿Se pueden explotar como canales laterales?

Es esencial abordar estas cuestiones para garantizar la seguridad de los datos sensibles y cumplir con las regulaciones pertinentes.

-----


## :snake: Vulnerabilidad SnakePro 

SnakePro es un juego en línea que representa una reinterpretación moderna del clásico juego de la serpiente. Permite a los usuarios registrarse y competir contra otros jugadores en tiempo real. Tiene integración con una base de datos MongoDB, que proporciona un almacenamiento de los datos del juego. Esta conexión a la base de datos permite la gestión de perfiles de usuario, puntuaciones y otros aspectos importantes del juego.

![Demo](https://hackmd.io/_uploads/H1ClMOr-C.png)

La infraestructura de la aplicación SnakePro está montada utilizando `Go (Golang)` como lenguaje principal. Go es un lenguaje de programación eficiente y altamente escalable que ofrece un conjunto completo de herramientas para desarrollar aplicaciones web y servicios de manera robusta y rápida. En este caso, la aplicación SnakePro hace uso de las bibliotecas y herramientas de Go para configurar y ejecutar servidores web, interactuar con bases de datos MongoDB, gestionar configuraciones, y más.

Entre algunas de las librerías encontramos:
```text!
os
fmt
net/http
html/template
io
errors
strconv
github.com/spf13/viper
context
strings
time
primitive
```

### Prueba de Concepto :mag:

Ahora que conocemos el propósito de esta aplicación, debemos entender cómo un atacante podría identificar y eventualmente encontrar información confidencial sobre la aplicación o sus usuarios.

La falta de cifrado al transmitir contraseñas en texto claro permite un ataque `man-in-the-middle`.

Al revisar cómo la aplicación almacena las contraseñas de los usuarios en MongoDB, fue posible ver que los datos sensibles se están almacenando en `texto claro`, como se puede ver en la función Register() (routes.go) y en la estructura UserData (types.go):

>:page_facing_up: routes.go

![image](https://hackmd.io/_uploads/BksZb90ZC.png)

>:page_facing_up: types.go

![image](https://hackmd.io/_uploads/H1OxXcCZC.png)

Además, el canal que los usuarios utilizan para enviar sus datos sensibles es inseguro (HTTP):

![image](https://hackmd.io/_uploads/SJojm5AZR.png)


Si la base de datos queda expuesta de alguna manera, se filtrarán las contraseñas de todos los usuarios:

![image](https://hackmd.io/_uploads/Hk7TV90bR.png)

----

## Solución :lock:

### Alto Nivel

Para abordar la vulnerabilidad en Snake Pro, de auerdo con [OWASP Top 10](https://owasp.org/Top10/es/A02_2021-Cryptographic_Failures/), A02 - Fallas Criptográficas aparatado `Cómo Prevenir`, se han implementado dos medidas de seguridad importantes:



- Hashing de contraseñas con `bcrypt`: En lugar de almacenar las contraseñas en texto plano, se utiliza la función de hash bcrypt para convertirlas en hashes seguros antes de almacenarlas en la base de datos. Esto significa que las contraseñas de los usuarios ya no están expuestas en su forma original, lo que hace más difícil para un atacante comprometer la seguridad del sistema mediante el acceso a las contraseñas.
- Uso de `TLS` para cifrar la comunicación: Se ha agregado un certificado TLS (anteriormente conocido como SSL) al servidor web de Snake Pro. Esto permite que todas las comunicaciones entre el navegador de los usuarios y el servidor web estén cifradas y sean seguras. La implementación de TLS protege la privacidad y la integridad de los datos transmitidos, lo que hace mucho más difícil para un atacante interceptar y leer la información confidencial, como las contraseñas o los datos de la sesión de los usuarios.


### Puesta en Marcha

| Archivo                                                           | Cambios                      | Descripción                                                                                                |
| ----------------------------------------------------------------- | ---------------------------- | ---------------------------------------------------------------------------------------------------------- |
| compose-dev.yaml                                                  | 11 adiciones                 | Se agregó un servicio Docker para el entorno de desarrollo con enlace de volumen para el socket de Docker. |
| owasp-top10-2021-apps/a2/snake-pro/Makefile                       | 2 adiciones                  | Se agregaron comandos para generar un certificado SSL y comenzar Docker Compose con configuración SSL.     |
| owasp-top10-2021-apps/a2/snake-pro/app/api/routes.go              | 19 adiciones                 | Se agregaron funciones para el cifrado de contraseñas, registro e inicio de sesión usando bcrypt.          |
| owasp-top10-2021-apps/a2/snake-pro/app/cert/certificate.crt       | 1 adición                    | Se agregó el archivo de certificado SSL.                                                                   |
| owasp-top10-2021-apps/a2/snake-pro/app/cert/certificate.key       | 1 adición                    | Se agregó el archivo de clave privada SSL.                                                                 |
| owasp-top10-2021-apps/a2/snake-pro/app/main.go                    | 2 adiciones, 2 eliminaciones | Se modificó para iniciar el servidor  con configuración SSL.                                           |
| owasp-top10-2021-apps/a2/snake-pro/app/views/form.html            | 3 adiciones, 3 eliminaciones | Se cambiaron los puertos de la URL en el formulario HTML.                                |
| owasp-top10-2021-apps/a2/snake-pro/deployments/check-init.sh      | 1 adición, 1 eliminación     | Se cambió el puerto a 443 para HTTPS y se modificaron los comandos curl en consecuencia.                   |
| owasp-top10-2021-apps/a2/snake-pro/deployments/docker-compose.yml | 1 adición, 1 eliminación     | Se cambió el mapeo de puerto a 443 para HTTPS.                                                             |

Una vez aplicadas las modificaciones, comprobamos que se han realizado satisfactoriamente.

- Contraseña hasheada en la base datos:
 ![image](https://hackmd.io/_uploads/S1Wl9JxzR.png)
 
- Transmisión de datos cifrada:
 ![image](https://hackmd.io/_uploads/HJoB51gfA.png)

#### Riesgos

Almacenar contraseñas en forma hasheada es una práctica de seguridad estándar para proteger la información de los usuarios en una base de datos. Sin embargo, esto implica que, en caso de olvido, `la contraseña original no se puede recuperar fácilmente`, lo que puede resultar `frustrante para los usuarios.` Los sistemas bien diseñados ofrecen métodos alternativos para restablecer contraseñas, como enlaces de restablecimiento por correo electrónico, evitando así comprometer la seguridad. Aunque esta técnica es segura, algunos usuarios pueden sentirse inquietos al saber que sus contraseñas se almacenan de manera irreversible, lo que podría generar preocupaciones sobre la privacidad y la seguridad de sus datos, impactando en la confianza en el sistema.

#### Vuelta Atrás

Para facilitar el proceso de reversión, se ha dejado un archivo comprimido llamado `snake-pro-old.tar.gz` en la misma ruta que el proyecto. Al descomprimir este archivo, se restaurará el estado anterior del proyecto. Una vez descomprimido, simplemente se ejecuta el comando `make install` para reinstalar el proyecto en su estado original. Este enfoque asegura una reversión rápida y sencilla a una versión estable y conocida del pr oyecto en caso de ser necesario.

![image](https://hackmd.io/_uploads/HJ5EdyxGC.png)
