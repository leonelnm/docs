# Básico

## Ciclo de vida de una entidad

### Estados de una entidad JPA

* `transient` (nueva): la entidad acaba de ser creada (posiblemente con el operador `new`) y aun no está asociada al contexto de persistencia. No tiene representación en la base de datos.
* `managed`, `persistent`: la entidad tiene un identificador y está asociada al contexto de persistencia. Puede estar almacenada en la base de datos, o aun no.
* `detached`: la entidad tiene un identificador, pero no está asociada al contexto de persistencia (normalmente), porque hemos cerrado el contexto de persistencia.
* `removed`: la entidad tiene un identificador y está asociada al contexto de persistencia, pero este tiene programada su eliminación.

## Tipos embebidos

Aunque se traten los valores como un objeto independiente, para la base de datos relacional, la información será guardada a modo de columnas.

En ocasiones, nos puede interesar tratar un grupo de atributos como si fueran uno solo. Un ejemplo clásico suele ser la dirección (nombre de la vía, número, código postal, …). Para este tipo de situaciones tenemos la posibilidad de covertir una clases en Embeddable. Veamos un ejemplo:

```java
@Embeddable
public class Direccion {

  @Column(nullable=false)
  private String via;

  @Column(nullable=false, length = 5)
  private String codigoPostal;

  @Column(nullable=false)
  private String poblacion;

  @Column(nullable=false)
  private String provincia;

  //...
}
```

```java
@Entity
@Table(name="USERCONEMBEDD")
public class User {

    @Id
    @GeneratedValue(strategy=GenerationType.AUTO)
    private long id;

    private String name;

  @Temporal(TemporalType.DATE)
    private Date birthDate;

  private Direccion address;

    //...
}
```

De esta forma, Hibernate detecta que el campo Direccion es una clase Embeddable, y mapea las columnas a la tabla USER.

### Sobrescritura con `@Embedded`

¿Qué pasaría si quisiéramos añadir dos direcciones a un usuario? Hibernate nos lanzará un error, indicando que no se soportan columnas con nombre repetido.

La solución la podemos aportar sobrescribiendo los atributos de la clase embebida, para que tengan otro nombre (o incluso otras propiedades)

```java
@Entity
@Table(name="USERCONEMBEDD")
public class User {

    @Id
    @GeneratedValue(strategy=GenerationType.AUTO)
    private long id;

  //Otros atributos

    @Embedded
    @AttributeOverrides({
        @AttributeOverride(name = "via", column = @Column(name="VIA_FACTURACION")),
        @AttributeOverride(name = "codigoPostal", column = @Column(name="CODIGOPOSTAL_FACTURACION", length=5)),
        @AttributeOverride(name = "poblacion", column = @Column(name="POBLACION_FACTURACION")),
        @AttributeOverride(name = "provincia", column = @Column(name="PROVINCIA_FACTURACION"))

    })
    private Direccion billingAddress;


  //...
```

* La anotación `@Embedded` es util cuando queremos mapear otras clases.&#x20;
* El atributo `@AttributeOverrides` selecciona las propiedades que serán sobrescritas.&#x20;
* El atributo `@AttributeOverride` indica el cambio que va a haber en un determinado atributo.

## Asociaciones entre entidades

### **Asociaciones Many-To-One `@ManyToOne`**

Conocida en algunos contextos como una relación padre/hijo, donde el lado muchos es el hijo y el lado uno es el padre.

```java
@Entity
public class Person {

    @Id
    @GeneratedValue
    private long id;

    private String name;

    public Person() { }

    public Person(String name) {
        this.name = name;
    }

    public long getId() {
        return id;
    }

    //resto de métodos...

}


@Entity
public class Phone {

    @Id
    @GeneratedValue
    private long id;

    private String number;

    @ManyToOne
    @JoinColumn(name = "person_id",
                    foreignKey = @ForeignKey(name="PERSON_ID_FK"))
    private Person person;


    public Phone() { }

    public Phone(String number) {
        this.number = number;
    }

    public String getNumber() {
        return number;
    }

    public void setNumber(String number) {
        this.number = number;
    }

    public Person getPerson() {
        return person;
    }

    public void setPerson(Person person) {
        this.person = person;
    }

    public long getId() {
        return id;
    }


}
```

### **Asociaciones One-To-Many `@OneToMany`**

```java
@Entity
public class Phone {

    @Id
    @GeneratedValue
    private long id;

    private String number;

    @ManyToOne
    private Person person;

    public Phone() { }

    //Resto de métodos

    @Override
    public int hashCode() {
        return Objects.hash(number);
    }

    @Override
    public boolean equals(Object o) {
        if ( this == o ) {
            return true;
        }
        if ( o == null || getClass() != o.getClass() ) {
            return false;
        }
        Phone phone = (Phone) o;
        return Objects.equals( number, phone.number );
    }


}

@Entity
public class Person {

    @Id
    @GeneratedValue
    private long id;

    @OneToMany(mappedBy = "person", cascade = CascadeType.ALL, orphanRemoval = true)
    private List<Phone> phones = new ArrayList<>();

    private String name;

    public Person() {
    }

    public Person(String name) {
        this.name = name;
    }

    public long getId() {
        return id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public List<Phone> getPhones() {
        return phones;
    }

    public void addPhone(Phone phone) {
        phones.add(phone);
        phone.setPerson(this);
    }

    public void removePhone(Phone phone) {
        phones.remove(phone);
        phone.setPerson(null);
    }

}
```

Desde `main`

```java
EntityManagerFactory emf = Persistence.createEntityManagerFactory("edu.home.hibernate");

EntityManager em = emf.createEntityManager();

em.getTransaction().begin();

Person p = new Person();
p.setName("rolem");

Phone phone1 = new Phone();
phone1.setPhoneNumber(989898);

Phone phone2 = new Phone();
phone2.setPhoneNumber(999999);

p.addPhone(phone1);
p.addPhone(phone2);

em.persist(p);
em.flush();

em.getTransaction().commit();

//Métodos de cierre...
```

### **Asociaciones One-To-One `@OneToOne`**

En una asociación uno-a-uno, solamente una instancia de una clase se asocia con una instancia de otra. Al usar el esquema unidireccional, tenemos que decidir un lado como **propietario**.

```java
@Entity
public class Phone {

    @Id
    @GeneratedValue
    private Long id;

    @Column(name = "`number`")
    private String number;

    @OneToOne(mappedBy = "phone", cascade = CascadeType.ALL, orphanRemoval = true, fetch = FetchType.LAZY)
    private PhoneDetails details;

    public Phone() {
    }

    //Resto de métodos

    public PhoneDetails getDetails() {
        return details;
    }

    public void addDetails(PhoneDetails details) {
        details.setPhone(this);
        this.details = details;
    }

    public void removeDetails() {
        if (details != null) {
            details.setPhone(null);
            this.details = null;
        }
    }
}


@Entity
public class PhoneDetails {

        @Id
        @GeneratedValue
        private Long id;

        private String provider;

        private String technology;

        @OneToOne(fetch = FetchType.LAZY)
        @JoinColumn(name = "phone_id")
        private Phone phone;

        public PhoneDetails() {
        }

            //Resto de métodos
}
```

### **Asociaciones Many-To-Many `@ManyToMany`**

*   **`@ManyToMany` Unidireccional**

    Tendremos que definir qué lado es el propietario de la asociación. En esa clase, incluimos una lista de elementos de la clase opuesta.

```java
@Entity
public class Person {

    @Id
    @GeneratedValue
    private Long id;
    @ManyToMany(cascade = {CascadeType.PERSIST, CascadeType.MERGE})
    private List<Address> addresses = new ArrayList<>();

    public Person() {
    }

    public List<Address> getAddresses() {
        return addresses;
    }

}

@Entity
public class Address {

    @Id
    @GeneratedValue
    private Long id;

    private String street;

    private String number;

    public Address() {
    }

    public Address(String street, String number) {
        this.street = street;
        this.number = number;
    }

    public Long getId() {
        return id;
    }

    public String getStreet() {
        return street;
    }

    public String getNumber() {
        return number;
    }

}
```

*   **`@ManyToMany` Bidireccional**

    Una asociación bidireccional `@ManyToMany` tiene un lado propietario y un lado `mappedBy`. Para preservar la sincronización entre ambos, es buena práctica añadir métodos helper para manejar las asociaciones.

```java
@Entity
public class Person {

    @Id
    @GeneratedValue
    private Long id;

    @NaturalId
    private String registrationNumber;
    @ManyToMany(cascade = {CascadeType.PERSIST, CascadeType.MERGE})
    private List<Address> addresses = new ArrayList<>();

    public Person() {
    }

    public Person(String registrationNumber) {
        this.registrationNumber = registrationNumber;
    }

    public List<Address> getAddresses() {
        return addresses;
    }

    public void addAddress(Address address) {
        addresses.add( address );
        address.getOwners().add( this );
    }

    public void removeAddress(Address address) {
        addresses.remove( address );
        address.getOwners().remove( this );
    }

    @Override
    public boolean equals(Object o) {
        if ( this == o ) {
            return true;
        }
        if ( o == null || getClass() != o.getClass() ) {
            return false;
        }
        Person person = (Person) o;
        return Objects.equals( registrationNumber, person.registrationNumber );
    }

    @Override
    public int hashCode() {
        return Objects.hash( registrationNumber );
    }

}

@Entity
public class Address {

    @Id
    @GeneratedValue
    private Long id;

    private String street;

    private String number;

    private String postalCode;

    @ManyToMany(mappedBy = "addresses")
    private List<Person> owners = new ArrayList<>();

    public Address() {
    }

    public Address(String street, String number, String postalCode) {
        this.street = street;
        this.number = number;
        this.postalCode = postalCode;
    }

    public Long getId() {
        return id;
    }

    //Resto de métodos

    @Override
    public boolean equals(Object o) {
        if ( this == o ) {
            return true;
        }
        if ( o == null || getClass() != o.getClass() ) {
            return false;
        }
        Address address = (Address) o;
        return Objects.equals( street, address.street ) &&
                Objects.equals( number, address.number ) &&
                Objects.equals( postalCode, address.postalCode );
    }

    @Override
    public int hashCode() {
        return Objects.hash( street, number, postalCode );
    }

}
```

