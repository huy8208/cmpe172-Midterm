# CMPE172 - Midterm part 2


#  1.0 Code Changes & Discussion 

StackEdit stores your files in your browser, which means all your files are automatically saved locally and are accessible **offline!**

## Explain (with code snippets from sample code) how the Web UI is able to remember" the selected Store.

The Web UI is able to remember the selected store with this Javascript code:
~~~

<script type="text/javascript">
        function selectedStore() {
            reg = document.getElementById("register").value
            switch (reg) {
                case "5012349":
                    document.getElementById("stores").options[0].selected = true
                    break;
                case "1287612":
                    document.getElementById("stores").options[1].selected = true
                    break;
                case "6498234":
                    document.getElementById("stores").options[2].selected = true
                    break;
                case "7812386":
                    document.getElementById("stores").options[3].selected = true
                    break;
                case "8723098":
                    document.getElementById("stores").options[4].selected = true
                    break;
                case "9621043":
                    document.getElementById("stores").options[5].selected = true
                    break;
                case "1393478":
                    document.getElementById("stores").options[6].selected = true
                    break;
                default:
                    document.getElementById("stores").options[0].selected = true
            }
        }
    </script>

~~~
When the page loads, the `selectedStore` function is called by the `onload` attribute on the `body` tag:


`<body  onload="selectedStore()">`

The `selectedStore` function reads the value of the "register" hidden input field:

`reg =  document.getElementById("register").value`

Next, the function uses a switch statement to check the value of `reg` and set the corresponding option in the "stores" dropdown as selected:

`switch  (reg) {  case  "5012349":  document.getElementById("stores").options[0].selected  =  true  break;  // other cases...  default:document.getElementById("stores").options[0].selected  =  true  }`

By doing this, when the page loads, the dropdown will display the appropriate store based on the "register" value. This way, the Web UI can "remember" the selected store across different requests.


## Explain (with code snippets from your code) how you implemented storing the Order in MySQL.

I have utilized the given Starbuck-api application from starbuck project. Using JPA dependency in the file. StarbucksOrder.java

~~~
@Entity
@Table(name  =  "STARBUCKS_ORDER")
@Data
@RequiredArgsConstructor
public  class  StarbucksOrder {
private @Id
@GeneratedValue
@JsonIgnore  /* https://www.baeldung.com/jackson-ignore-properties-on-serialization */
Long id;
@Column(nullable  =  false)
private  String drink;
@Column(nullable  =  false)
private  String milk;
@Column(nullable  =  false)
private  String size;
private  double total;
private  String status;
private  String register;
@OneToOne(cascade  = CascadeType.ALL)
@JoinColumn(name  =  "card_id", referencedColumnName  =  "id")
@JsonIgnore  /* https://www.baeldung.com/jackson-ignore-properties-on-serialization */
private  StarbucksCard card;
~~~

Then making client call to REST functions in StarbucksOrderController.java will save the order to database with 

~~~
StarbucksOrder new_order = ordersRepository.save(order);
~~~

## Explain (with code snippets from your code) how you support generating a different "random" order with each "Place Order" request (instead of the starter code's "hard coded" order)

This is how I generate random order
~~~
private Order generateRandomOrder() {
        String[] DRINK_OPTIONS = { "Caffe Latte", "Caffe Americano", "Caffe Mocha", "Espresso", "Cappuccino" };
        String[] MILK_OPTIONS = { "Whole Milk", "2% Milk", "Nonfat Milk", "Almond Milk", "Soy Milk" };
        String[] SIZE_OPTIONS = { "Short", "Tall", "Grande", "Venti", "Your Own Cup" };
        Random random = new Random();
        String randomDrink = DRINK_OPTIONS[random.nextInt(DRINK_OPTIONS.length)];
        String randomMilk = MILK_OPTIONS[random.nextInt(MILK_OPTIONS.length)];
        String randomSize = SIZE_OPTIONS[random.nextInt(SIZE_OPTIONS.length)];
        Order randomOrder = new Order();
        randomOrder.setDrink(randomDrink);
        randomOrder.setMilk(randomMilk);
        randomOrder.setSize(randomSize);
        randomOrder.setStatus("Ready for Payment");
        // Set price based on drink and size
        double price;
        switch (randomDrink) {
            case "Caffe Latte":
                price = (randomSize.equals("Tall") ? 2.95 : randomSize.equals("Grande") ? 3.65 : 3.95);
                break;
            case "Caffe Americano":
                price = (randomSize.equals("Tall") ? 2.25 : randomSize.equals("Grande") ? 2.65 : 2.95);
                break;
            case "Caffe Mocha":
                price = (randomSize.equals("Tall") ? 3.45 : randomSize.equals("Grande") ? 4.15 : 4.45);
                break;
            case "Espresso":
                price = (randomSize.equals("Short") ? 1.75 : 1.95);
                break;
            default: // "Cappuccino"
                price = (randomSize.equals("Tall") ? 2.95 : randomSize.equals("Grande") ? 3.65 : 3.95);
                break;
        }
        double tax = 0.0725;
        double total = price + (price * tax);
        double scale = Math.pow(10, 2);
        double rounded = Math.round(total * scale) / scale;
        randomOrder.setTotal(rounded);
        return randomOrder;
~~~

## Explain (with code snippets from your code) how you added support for Spring Security / Login Page

To add Spring Security and a login page to my application, you used the `HttpSecurity` object in the `configure` method of my `WebSecurityConfig` class, which extends `WebSecurityConfigurerAdapter`.

Here's a step-by-step explanation of my code:
1.  Configure authorization rules with `http.authorizeRequests()`:
    `http.authorizeRequests()  // URL matching for accessibility  .antMatchers("/",  "/auth/login.css",  "/login",  "/register",  "/h2-console/**") .permitAll() .antMatchers("/admin/**").hasAnyAuthority("ADMIN") .antMatchers("/user/**").hasAnyAuthority("USER") .anyRequest().authenticated()`
    This code sets the access rules for different URL patterns:
    -   Allows unauthenticated access to the root path, the login page, the register page, and the H2 console.
    -   Allows access to `/admin/**` URLs only for users with the "ADMIN" authority.
    -   Allows access to `/user/**` URLs only for users with the "USER" authority.
    -   Requires authentication for any other request.
2.  Configure form-based login:
    // form login with default login page
    `.and() .formLogin().successHandler(sucessHandler)`
    This code enables the default form-based login and specifies a custom success handler (`sucessHandler`), which handles login success events and redirects users to the appropriate page based on their roles.

3.   What's the User Name and Password you created? How did you "Hash" the Password?

- Username: hello
- Password: dancing
- This is how I hash the password:

~~~
   @Bean
    public InMemoryUserDetailsManager inMemoryUserDetailsManager() {
        InMemoryUserDetailsManager userDetailsManager = new InMemoryUserDetailsManager();
        userDetailsManager.createUser(hardcodedUser());
        return userDetailsManager;
    }

    private UserDetails hardcodedUser() {
        return User.withUsername("hello")
                .password(passwordEncoder().encode("dancing"))
                .authorities("USER")
                .build();
    }
   ~~~

## How does your implementation work behind a Load Balancer without using LB Sticky Sessions?

Setting up Redis in application.properties

~~~
spring.session.store-type=redis 
spring.redis.host=${REDIS_HOST:localhost}
spring.redis.password=${REDIS_PASSWORD:foobared}
spring.redis.port=${REDIS_PORT:6379}
~~~
And in docker-compose

~~~
  redis:
    image: redis
    platform: linux/arm64/v8
    networks:
      - network
    ports:
      - "6379:6379"
    restart: always
~~~

## How does your solution managed "remembering" the active order for a register / store?

Again, since I have utilized Starbuck-api, the backend REST function automatically return "active order" if an order has already been placed.

- This is part of the newOrder function that can be found in StarbucksService.java under Starbuck-Api, it will throw error 
>StarbucksService.java
~~~
    public StarbucksOrder newOrder(String regid, StarbucksOrder order) throws ResponseStatusException {
        System.out.println("Placing Order (Reg ID = " + regid + ") => " + order);
        // check input
        if (order.getDrink().equals("") || order.getMilk().equals("") || order.getSize().equals("")) {
            throw new ResponseStatusException(HttpStatus.BAD_REQUEST, "Invalid Order Request!");
        }
        // check for active order
        StarbucksOrder active = orders.get(regid);
        if (active != null) {
            System.out.println("Active Order (Reg ID = " + regid + ") => " + active);
            if (active.getStatus().equals("Ready for Payment."))
                throw new **ResponseStatusException(HttpStatus.BAD_REQUEST, "Active Order Exists!"**);
        }
~~~

- Then in my client Rest function, I can catch the ResponseStatusException
> StarbucksController.java
~~~
ApiResponse result = createNewOrder(command.getRegister(), orderRequest);
            if (result.isSuccess()) {
                log.info("The order was created successfully.");
                System.out.println(result.getData());
            } else {
                if ("Active Order Exists!".equals(result.getMessage())) {
                    message = "Error: Active order already exists for this register!";
                }
                log.error(result.getMessage());
            }
~~~

## Did you make any System Architecture Changes to support the requirements?

Yes I did, I added Starbucks-Api to handle backend REST request and also save orders to MYSQL. 
Please check the System Architecture Diagram below

## Note

- Please ignore that I have connected Spring-Cashier app to MySql, this design is not finalized.
- Since I am using Mac M1, I have specified linux/arm64/v8 as my preferred platform since it speeds up my coding.
# **2.0 Successful Deployment**

> Dockerfile
~~~
FROM openjdk:11
EXPOSE 8080
ADD ./target/springcashier-1.0.jar /srv/springcashier-1.0.jar
CMD java -jar /srv/springcashier-1.0.jar
~~~

>Docker-compose.yaml
~~~

version: "3"

services:
  mysql:
    image: mysql:8.0
    platform: linux/arm64/v8
    volumes:
      - /tmp:/tmp
      - ./init.sql:/docker-entrypoint-initdb.d/init.sql
    ports:
      - 3306
    networks:
      - network
    environment:
      MYSQL_ROOT_PASSWORD: "cmpe172"
    restart: always     
  starbucks:
    image: spring-starbucks-api
    platform: linux/arm64/v8
    depends_on:
    - mysql    
    volumes:
      - /tmp:/tmp
    networks:
      - network   
    ports:
      - 8080
    environment:
      MYSQL_HOST: "mysql"
      MYSQL_USER: "firstuser"
      MYSQL_PASS: "hello"
    restart: always
  spring-cashier:
    image: spring-cashier
    platform: linux/arm64/v8
    volumes:
      - /tmp:/tmp
    networks:
      - network
    depends_on:
      - mysql
    ports:
      - 9090 #local dev
      # - 90:9090 #local dev
    environment:
      MYSQL_HOST: "mysql"
      MYSQL_USER2: "seconduser"
      MYSQL_PASS2: "goodbye"
      API_HOST: "starbucks:8080"
      API_KEY: "2H3fONTa8ugl1IcVS7CjLPnPIS2Hp9dJ"
      API_ENDPOINT: "http://spring-starbucks-api:8080"
      REDIS_HOST: "redis"
      REDIS_PASSWORD: "foobared"
    restart: always 
  lb:
    image: eeacms/haproxy
    platform: linux/amd64
    depends_on:
    - spring-cashier
    ports:
    - "80:5000"
    - "1936:1936"
    environment:
      BACKENDS: "spring-cashier"
      BACKENDS_PORT: "8080"
      DNS_ENABLED: "true"
      COOKIES_ENABLED: "false"
      LOG_LEVEL: "info"
    networks:
      - network
  redis:
    image: redis
    platform: linux/arm64/v8
    networks:
      - network
    ports:
      - "6379:6379"
    restart: always
volumes:
  schemas:
    external: false

networks:
  network:
    driver: bridge


~~~

>Spring-cashier application.properties 
~~~
server.error.include-message=always
server.error.include-exception=true
server.error.include-stacktrace=always
server.error.include-binding-errors=always

# MYSQL
spring.jpa.hibernate.ddl-auto=update
spring.datasource.url=jdbc:mysql://${MYSQL_HOST:localhost}:3306/midterm
spring.datasource.username=${MYSQL_USER2:seconduser}
spring.datasource.password=${MYSQL_PASS2:goodbye}

# Starbuck-Api
starbucks.client.apikey=${API_KEY:2H3fONTa8ugl1IcVS7CjLPnPIS2Hp9dJ}
starbucks.client.apihost=${API_HOST:localhost:8080}

# Redis Session Store
spring.session.store-type=redis 
# Redis server host
spring.redis.host=${REDIS_HOST:localhost}
# Login password of the redis server
spring.redis.password=${REDIS_PASSWORD:foobared}
# Redis server port
spring.redis.port=${REDIS_PORT:6379}
~~~
## UML diagrams
![image](https://github.com/huy8208/cmpe172-Midterm/blob/main/Architecture%20Diagram.png)
