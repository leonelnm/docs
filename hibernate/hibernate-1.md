# Relaciones entre tablas

## Herencia y valores generados

La herencia es un mecanismo definido por la programación orientada a objetos, mediante el que se define una relación jerárquica entre varias clases.

JPA+Hibernate ofrecen diferentes mecanismos para trasladar esta relación jerárquica a base de datos en función de las necesidades.

### `@MappedSuperclass`

En este primer esquema, la clase base **no será trasladada a la base de datos.** Es decir, está disponible en la aplicación para trabajar con ella, pero no generará una nueva tabla.

No es posible hacer uso de consultas polimórficas, y por tanto se debe consultar ambas entidades por separado.

```java
@MappedSuperclass
public class Cuenta implements Serializable {

    @Id
    @GeneratedValue
    private long id;
    private String titular;
    private double balance;
    private double tipoInteres;
    private static final long serialVersionUID = 1L;

    public Cuenta() {
        super();
    }   
    public long getId() {
        return this.id;
    }

    public String getTitular() {
        return this.titular;
    }

    public void setTitular(String titular) {
        this.titular = titular;
    }   
    public double getBalance() {
        return this.balance;
    }

    public void setBalance(double balance) {
        this.balance = balance;
    }   
    public double getTipoInteres() {
        return this.tipoInteres;
    }

    public void setTipoInteres(double tipoInteres) {
        this.tipoInteres = tipoInteres;
    }
}
```

```java
@Entity
public class CuentaCredito extends Cuenta implements Serializable {


    private double limiteCredito;
    private static final long serialVersionUID = 1L;

    public CuentaCredito() {
        super();
    }   
    public double getLimiteCredito() {
        return this.limiteCredito;
    }

    public void setLimiteCredito(double limiteCredito) {
        this.limiteCredito = limiteCredito;
    }

}
```

```java
@Entity
public class CuentaDebito extends Cuenta implements Serializable {


    private double cargoPorDescubierto;
    private static final long serialVersionUID = 1L;

    public CuentaDebito() {
        super();
    }   
    public double getCargoPorDescubierto() {
        return this.cargoPorDescubierto;
    }

    public void setCargoPorDescubierto(double cargoPorDescubierto) {
        this.cargoPorDescubierto = cargoPorDescubierto;
    }

}
```

La clase **Cuenta** trabaja como si fuera una entidad, se debe definir dentro de la unidad de persistencia, pero no se traslada a la base de datos.

*   _El **DDL generado** por Hibernate sería el siguiente:_

    \`\`\`sql

    create table CuentaCredito (

    ```
     id bigint not null,
      balance double precision not null,
      tipoInteres double precision not null,
      titular varchar(255),
      limiteCredito double precision not null,
      primary key (id)
    ```

    &#x20; ) engine=InnoDB

create table CuentaDebito ( id bigint not null, balance double precision not null, tipoInteres double precision not null, titular varchar(255), cargoPorDescubierto double precision not null, primary key (id) ) engine=InnoDB

````
Cada una de las entidades extendidas reciben todos los atributos de la entidad base (*Cuenta*).

## `Single Table`
>*`@Inheritance(strategy = InheritanceType.`**SINGLE_TABLE**)*

Con la anotación `@SingleTable`, se indica a Hibernate que para todas las entidades que participen en la jerarquía de herencia, solamente tiene que crear una tabla en base de datos.

```java
@Entity
@Inheritance(strategy = InheritanceType.SINGLE_TABLE)
public class Cuenta implements Serializable {

    @Id
    @GeneratedValue
    private long id;
    private String titular;
    private double balance;
    private double tipoInteres;
    private static final long serialVersionUID = 1L;

    public Cuenta() {
        super();
    }   
    public long getId() {
        return this.id;
    }

    public String getTitular() {
        return this.titular;
    }

    public void setTitular(String titular) {
        this.titular = titular;
    }   
    public double getBalance() {
        return this.balance;
    }

    public void setBalance(double balance) {
        this.balance = balance;
    }   
    public double getTipoInteres() {
        return this.tipoInteres;
    }

    public void setTipoInteres(double tipoInteres) {
        this.tipoInteres = tipoInteres;
    }

}
````

```java
@Entity
public class CuentaCredito extends Cuenta implements Serializable {

  //Igual que el ejemplo anterior

}
```

```java
@Entity
public class CuentaDebito extends Cuenta implements Serializable {

  //Igual que el ejemplo anterior

}
```

El **DDL generado** por Hibernate, es el siguiente:

```sql
create table Cuenta (

    DTYPE varchar(31) not null,
    id bigint not null,
    balance double precision not null,
    tipoInteres double precision not null,
    titular varchar(255),
    cargoPorDescubierto double precision,
    limiteCredito double precision,
    primary key (id)

) engine=InnoDB
```

Ha añadido una columna, llamada _**DTYPE**_, que actúa de discriminante. Para el ejemplo:

```java
CuentaCredito cuentaCredito = new CuentaCredito();
cuentaCredito.setTitular("Luismi");
cuentaCredito.setBalance(500.0);
cuentaCredito.setTipoInteres(0.20);
cuentaCredito.setLimiteCredito(600.0);


CuentaDebito cuentaDebito = new CuentaDebito();
cuentaDebito.setTitular("Luismi");
cuentaDebito.setBalance(200.0);
cuentaCredito.setTipoInteres(0.10);
cuentaDebito.setCargoPorDescubierto(6.5);

em.persist(cuentaCredito);
em.persist(cuentaDebito);
```

Se obtiene el resultado: ![](https://via.placeholder.com/468x60)

En cada fila ha añadido como _DTYPE_ el nombre de la clase entidad. Este valor de discriminante se puede modificar mediante la anotación `@DiscriminatorValue`

```java
@Entity
@DiscriminatorValue("newName")
public class CuentaCredito extends Cuenta implements Serializable {

  //Igual que el ejemplo anterior

}
```

```java
@Entity
@DiscriminatorValue("newName")
public class CuentaDebito extends Cuenta implements Serializable {

  //Igual que el ejemplo anterior

}
```

### `Joined Table`

> _`@Inheritance(strategy = InheritanceType.`**SINGLE\_TABLE**)_

Este esquema de trabajo, también conocido como _**table per subclass**_, genera las siguientes tablas:

* Una tabla para la entidad **base** de la jerarquía. Tendrá todos los atributos de la clase base.
* Una tabla para cada entidad extendida de la jerarquía. Tendrá una referencia a la entidad base, y los _atributos propios_.

```java
@Entity
@Inheritance(strategy = InheritanceType.JOINED)
public class Cuenta implements Serializable {

  //Igual que el ejemplo anterior

}
```

```java
@Entity
public class CuentaCredito extends Cuenta implements Serializable {

  //Igual que el ejemplo anterior

}
```

```java
@Entity
public class CuentaDebito extends Cuenta implements Serializable {

  //Igual que el ejemplo anterior

}
```

Esto genera un DDL en el que se obtienen 3 tablas, y 2 claves foraneas.

```sql
create table Cuenta (
    id bigint not null,
    balance double precision not null,
    tipoInteres double precision not null,
    titular varchar(255),
    primary key (id)
) engine=InnoDB

create table CuentaCredito (
    limiteCredito double precision not null,
    id bigint not null,
    primary key (id)
) engine=InnoDB

create table CuentaDebito (
    cargoPorDescubierto double precision not null,
    id bigint not null,
    primary key (id)
) engine=InnoDB

alter table CuentaCredito
    add constraint FK6641o76fphgs98cbv18sd7htc
    foreign key (id)
    references Cuenta (id)

alter table CuentaDebito
    add constraint FK2gigt3h95mq590key3xvkhqk0
    foreign key (id)
    references Cuenta (id)
```

A diferencia del esquema `@MappedSuperclass`, con este esquema si es posible usar consultas polimórficas, pero tener en cuenta que esto puede utilizar muchos JOIN, lo cual puede penalizar el rendimiento de la base de datos.

> Si se requiere un nombre diferente a las claves externas, usar la anotación `@PrimaryKeyJoinColumn(name = " ...")`.

```java
....

@Entity
@PrimaryKeyJoinColumn(name = "account_id")
public class CuentaCredito extends Cuenta implements Serializable {

  //Igual que el ejemplo anterior

}

....
```

## Campos calculados o derivados

### `@Generated`

Un campo derivado o calculado es un campo que no se gestiona igual que los demás: su valor viene a raíz de un cálculo a partir de, normalmente, el resto de campos de la entidad, u otro valor especial. Estos valores son generados por la base de datos (y no por Java). Cuando Hibernate detecta una inserción o actualización, realiza una (re)generación del valor del campo calculado.

Para poder anotar un campo como generado, usamos la anotación `@Generated`. Esta permite indicar cuando el campo será generado:

* INSERT: Será generado en la inserción, pero no en las sentencias UPDATE posteriores.
* ALWAYS: En cualquier modificación (INSERT, UPDATE).

Para indicar este valor, usamos GenerationTime como en el ejemplo siguiente:

```java
@Entity
@Table(name="ADVANCEDUSER")
public class User {

    @Id
    @GeneratedValue
    private long id;

    private String email;

    private String city;

    @Generated(value=GenerationTime.ALWAYS)
    @Column(columnDefinition=
        " varchar(512) AS (CONCAT(email,' (', city,')')) "
    )
    private String generated;

    public long getId() {
        return id;
    }

    public String getEmail() {
        return email;
    }

    public void setEmail(String email) {
        this.email = email;
    }

    public String getCity() {
        return city;
    }

    public void setCity(String city) {
        this.city = city;
    }

    public String getGenerated() {
        return this.generated;
    }
}
```

Como de la generación de este campo se encarga la base de datos, se debe indicar la forma en que lo hará al estilo SQL. (Depende de cada motor de DB)

Mediante `@Column(columnDefinition="...")` le estamos indicando el tipo de dato y como hacer el cálculo.

El DDL generado sería:

```sql
create table ADVANCEDUSER (
    id bigint not null,
    city varchar(255),
    email varchar(255),
    presentation varchar(512) AS (CONCAT(email,' (', city,')')),
    primary key (id)
) engine=InnoDB
```

### `@CreationTimestamp`

La anotación `@CreationTimestamp` permite indicar a Hibernate que en el atributo anotado con esta anotación debe almacenarse la actual fecha y hora de la JVM cuando la entidad sea almacenada.

```java
@Entity
@Table(name="ADVANCEDUSER")
public class User {

    @Id
    @GeneratedValue
    private long id;

    //Resto de atributos


    @CreationTimestamp
    private Date createdDate;

    public Date getCreatedDate() {
        return createdDate;
    }

    //Resto de métodos
}
```

### `@ColumnTransformer`

Hibernate permite personalizar el código SQL que utiliza para leer o almacenar los valores de algunas columnas.

Por ejemplo, si RDBMS tiene alguna función de encriptación o codificación, se puede invocar para almacenar una columna (por ejemplo, una contraseña).

```java
@Entity
@Table(name="ADVANCEDUSER")
public class User {


    //Resto de atributos

    @Column(name="pswd")
    @ColumnTransformer(
            write= " MD5(?) " //depende de cada motor de DB
    )
    private String password;

    //Resto de métodos

    public String getPassword() {
        return password;
    }

    public void setPassword(String password) {
        this.password = password;
    }
}
```
