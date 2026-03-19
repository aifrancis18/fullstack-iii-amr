#  Comunicación entre Microservicios con Spring Cloud OpenFeign

##  a. Descripción de esta forma de comunicación

En esta implementación se utiliza **Spring Cloud OpenFeign** para permitir la comunicación entre microservicios de forma sencilla y desacoplada.

OpenFeign es un cliente HTTP declarativo que permite a un microservicio consumir endpoints de otro microservicio como si fuera una llamada a un método Java, evitando el uso manual de herramientas como `RestTemplate`.

###  Funcionamiento en este proyecto

El sistema está compuesto por dos microservicios:

* **Servicio de Canchas**
* **Servicio de Reservas**

Cada uno posee su propia base de datos, lo que garantiza independencia y desacoplamiento.

###  Flujo de comunicación

1. El cliente realiza una petición para crear una reserva.

2. El microservicio de reservas recibe la solicitud.

3. Antes de guardar la reserva, valida que la cancha exista.

4. Para esto, utiliza OpenFeign para consumir el endpoint del microservicio de canchas:

   ```
   GET /api/canchas/{id}
   ```

5. Si la cancha existe, la reserva se guarda en su base de datos.

6. Si no existe, se lanza un error.

###  Características clave

* Comunicación basada en HTTP (REST)
* Uso de interfaces Java en lugar de código HTTP manual
* Bajo acoplamiento entre servicios
* Validación de datos entre microservicios
* Separación de responsabilidades

---

## 🛠️ b. Cambios que se deben implementar en el proyecto

Para implementar la comunicación con OpenFeign, se realizaron los siguientes cambios:

---

### 1. Agregar dependencia en `pom.xml`

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>
```

---

### 2. Habilitar Feign en la aplicación

En la clase principal del microservicio de reservas:

```java
@EnableFeignClients
@SpringBootApplication
public class ServicioreservasApplication {
    public static void main(String[] args) {
        SpringApplication.run(ServicioreservasApplication.class, args);
    }
}
```

---

### 3. Crear cliente Feign

Se crea una interfaz para consumir el microservicio de canchas:

```java
@FeignClient(name = "canchas-service", url = "http://localhost:8081")
public interface CanchaClient {

    @GetMapping("/api/canchas/{id}")
    Cancha obtenerCancha(@PathVariable("id") Long id);
}
```

---

### 4. Crear clase modelo `Cancha` en reservas

Se define un modelo simple para recibir los datos desde el otro microservicio:

```java
public class Cancha {

    private Long id;
    private String nombre;
    private String tipo;
    private Double precioPorHora;

    // getters y setters
}
```

> ⚠️ Esta clase no es una entidad (`@Entity`), solo se usa como DTO para comunicación.

---

### 5. Modificar lógica del servicio de reservas

Se integra el cliente Feign para validar la existencia de la cancha:

```java
@Override
public Reserva crearReserva(Reserva reserva) {

    Cancha cancha = canchaClient.obtenerCancha(reserva.getCanchaId());

    if (cancha == null) {
        throw new RuntimeException("La cancha no existe");
    }

    return reservaRepository.save(reserva);
}
```

---

### 6. Configuración de puertos

Cada microservicio debe ejecutarse en un puerto distinto:

#### Servicio Canchas

```
server.port=8081
```

#### Servicio Reservas

```
server.port=8082
```

---

### 7. Configuración de bases de datos separadas

Cada microservicio utiliza su propia base de datos:

* `club_deportivo`
* `club_deportivo2`

Esto permite:

* Independencia de datos
* Escalabilidad
* Bajo acoplamiento

---

##  Conclusión

La implementación de OpenFeign permite que los microservicios se comuniquen de forma clara, mantenible y escalable.

Se logra:

* Separación de responsabilidades
* Independencia de bases de datos
* Validación entre servicios
* Arquitectura basada en microservicios real

---

 Esta arquitectura es la base para sistemas distribuidos modernos.
