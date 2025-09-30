# ORM JDBC JPA Hibernate Spring Data
maaping objet relationnel (correspondance entre => approche `oriente objet`: creer des class ... +
`bases de donnees relationnel`: contenant les tableaux relationnel des donnees)

### EXEMPLE: rechercher des produits Image: 
![Produit Screenshot](images/app-oriente-objet.png)
1. creer class Produit:
```java
public class Produit(
	private Long id;
	private string designation;
	private double prix;
	private int quantite;
	// GETTERS SETTERS
)
```
2. creer table Produits contenant les donnees
| ID | Designation   | Prix | Quantite |
|----|---------------|------|----------|
| 1  | Clavier       | 150  | 12       |
| 2  | Souris Gamer  | 120  | 15       |
| 3  | Casque Audio  | 250  | 10       |

3. creer methode to connect between 1 and 2 we use `api jdbc` JAVA DATABSAE CONNECTIVITY

```java
public List<Produit> findByDesignation(String kw) {
	List<Produit> produits=new ArrayList<Produit>(); //connect between a table data and class
	Class.forName("com.mysql.jdbc.Driver"); //connect to database server MySQL using class/driver
	Connection conn=DriverManager.getConnection ("URL", "username", "psw");
	PreparedStatement ps=conn.preparedStatement("SELCT * FROM PRODUITS WHERE DISIGNATION LIKE ?");
	ps.serString(1, kw);
	ResultSet rs=ps.executeAuery(); //executer a reuqete - ResultSet used to stock results of select query
	while(rs.next()){
		Produit p=new Produit();
		p.setId(rs.getLong("ID"));
		p.setDesignation(rs.getString("DESIGNATION"));
		p.setPrix(rs.getDouble("PRIX"));
		p.setQuantite(rs.getint("QUANTITE"));
		produits.add(p):
	}
	return produits;
}
```

* if u did PreparedStatement ps=conn.preparedStatement("SELCT * FROM PRODUITS WHERE DISIGNATION LIKE" + "'" + "%" + kw + "%" + "'") there s a probability of sql injection to ur code (fail de securite) so u have to do tests concatination if kw contain DROP DLETE ... keywords so best practice is ps.serString(1, keyword)
* instead of using this heavy code we use a framework called HIBERNATE to do object relational mapping 

---

## WHY Hibernate?
- -95% of wasted coding time
- portability of app for example from mysql to oracle
- performance / best practice

all mapping frameworks follow JPA java persistence api used to create interfaces

### EXEMPLE: APP pour gerer les produits `Entity JPA Produit` / `Java Bean`
```java
package dao;

import java.io.Serializable;
import javax.persistence.*;

@Entity //means that this class corresponds to a table in the database which is called `persistent class`
@Table(name="PRODUITS")

public class Produit implements Serializable {
	@Id //means that reference is a primary key
	@GeneratedValue(strategy=GeneratedType.IDENDITY) //identity means incremented id by one
	@Column(name="REF") //means that the variable reference corresponds to column REF in database
	private Long reference;
	@Column(name="DES")
	private string designation;
	private double prix;
	private int quantite;

	//constructors 
	//getters setters
}
```
![Produit Screenshot](images/serialization.png)
* serialization prendre un objet de la ram/memoire et de le convertir en un tableau d octet `bits` pour l envoyer a une autre app after that the other application does deserialization prendre le taleau d octets pour construire l objet  
* par defaut entities are converted in format `XML` ou `JSON` between layers of different types/codes/languages

---

### Config unite de persistence XML:

precise `name` / `provider` (Hibernate) / `properties` => connection + username + pwd + driver_class (jdbc Driver) / `dialect` specified sql/oracle ... version / `hbm2ddl` hibernate mapping to data definition language - theres also instead of ddl dml (data manipulation language) or dcl (data control language)

* CAREFUL IN hbm2ddl to do value create cause that we ll delete already existing tables / create is only used in developppemnt so u have to separate u re project to 4 environments: `developpement`, `test`, `pre-production`, `production`

### Ajouter un produit: methode persist()
-> NORMAL JPA:
```java
public void save(Produit p) {
	EntityTransaction transaction=entityManager.getTransaction(); //create transaction
	transaction.begin(); //start transaction
	try {
		entityManager.persist(p); //register product datas in database
		transaction.commit(); //validate transaction
	} catch (Exception e) {
		transaction.rollback();
		e.printStackTrace();
	}
}
```
-> Spring Boot:
```java
@Transactional //spring method to manage transactions
public void save(Produit p) {
	entityManager.persist(p);
}
```

### Consulter les produits: createQuery()
```java
public List<Produit> findAll() {
	Query query=entityManager.createQuery("select p from Produit p"); //it s not sql it s hql hibernate query languge
	return query.getResultList();
}
```
* `HQL` used to manipulate relations between classes and relation classes u can use instead of `createQuery` - `createSQLQuery` which is bad practice

### Consulter les produits par mot cle: createQuery()
```java
public List<Produit> findByDesignation(String kw) {
	Query query=entityManager.createQuery("select p from Produit p where p.designation like :x"); //it s not sql it s hql hibernate query languge
	query.setParameter("x" + "%" + kw + "%" + "'");
	return query.getResultList();
}
```

### Consulter un produits: methode find()
```java
public Produit findById(Long idProduit) {
	Produit p=entityManager.find(Produit.class, idProduit);
	return p;
}
```

### Mettre a jour un produits: methode merge()
```java
public void update(Produit p) {
	entityManager.merge(p);
}
```

### Supprimer un produits: methode remove()
```java
public void deleteById(Long idProduit) {
	Produit p=entityManager.find(Produit.class, idProduit);
	entityManager.remove(p);
}
```

to save a client instead of repeating this:
```java
@Transactional
public void save(Client c) {
	entityManager.persist(c);
}
```
a better practice is to do this:
```java
public void save<T>(T t) {
	entityManager.persist(t);
}
```
but and even better practice is to create an interface:
```java
public class Repository<T, U> {
	public T save(T o) {
		entityManager.persist(t);
	}
	public List<T> findAll() {
		//code
	}
	//other methods ...
}
```
=> So spring data has the role to create a class that has many methods that can be used by ur entities
