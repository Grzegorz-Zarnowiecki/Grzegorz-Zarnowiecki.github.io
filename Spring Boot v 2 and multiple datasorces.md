# Spring Boot v 2 and multiple datasorces 

Hello 

Finally i just read documentation and somme blogs, after merging proces I wrtite this code...:

```java

@Configuration
@EnableTransactionManagement
@EnableJpaRepositories(basePackages = "net.gzarnowiecki.multids.H2.repository", entityManagerFactoryRef = "h2DSEmFactory", transactionManagerRef = "h2DSTransactionManager")

public class H2DbConfiguration {
    @Value( "${h2.datasource.hibernatedialect}" )
    private String hibernatedialect;
    @Value( "${h2.datasource.hibernate.hbm2ddl.auto}" )
    private String hbm2ddl;

    @Primary
    @Bean
    @ConfigurationProperties("h2.datasource")

    public @Qualifier("h2.datasource")
    DataSourceProperties h2DSProperties() {
        DataSourceProperties ds = new DataSourceProperties();
        return ds;
    }

    @Primary
    @Bean
    public DataSource h2DS(@Qualifier("h2DSProperties") DataSourceProperties h2DSProperties) {
        return h2DSProperties.initializeDataSourceBuilder().build();
    }

    @Primary
    @Bean
    public LocalContainerEntityManagerFactoryBean h2DSEmFactory(@Qualifier("h2DS") DataSource h2DS, EntityManagerFactoryBuilder builder) {
        Properties properties = new Properties();
        properties.setProperty("hibernate.dialect", hibernatedialect);
        properties.setProperty("hibernate.hbm2ddl.auto","create");

        LocalContainerEntityManagerFactoryBean emf = builder.dataSource(h2DS).packages("net.gzarnowiecki.multids.H2.domain").build();
        emf.setJpaProperties(properties);
        emf.setPersistenceUnitName("h2Presistence");
        return emf;
    }

    @Primary
    @Bean
    public PlatformTransactionManager h2DSTransactionManager(@Qualifier("h2DSEmFactory")EntityManagerFactory h2DSEmFactory) {
        return new JpaTransactionManager(h2DSEmFactory);
    }

}
```

This i configuration for first data source. When You create another please remove @Primary annotation.

and this  part of configuration **application.yml**

```ini
h2:
  datasource:
    url: jdbc:h2:mem:mydb
    username: sa
    password:
    driverClassName: org.h2.Driver
    connection-test-query: SELECT 1
    hibernatedialect: org.hibernate.dialect.H2Dialect
    hibernate.hbm2ddl.auto: create-drop
```

