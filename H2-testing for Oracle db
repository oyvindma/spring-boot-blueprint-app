Put in src/main to be able to start H2 with support for 0/1 instead of true/false
Connection string: jdbc:h2:./lokaldb;MODE=Oracle;DEFAULT_NULL_ORDERING=HIGH;AUTO_SERVER=TRUE

```
@Configuration
class DataSourceConfig(val dataSourceProperties: DataSourceProperties) {
    final val log: Logger = LoggerFactory.getLogger(this::class.java)
    init {

        if (dataSourceProperties.driverClassName == "org.h2.Driver") {
            /*
                For å få tilstrekkelig kompatibilitet med Oracle ved kjøring mot H2 må
                Mode.numericWithBooleanComparison=true bli satt.

                Det er ikke mulig å kode direkte mot H2 sine klasser siden det primært er en test-dependency.
                MEN, når applikasjonen blir startet via IntelliJ eller mvn spring-boot:run blir H2 automagisk lagt til
                på classpath for at applikasjonen skal kunne kjøre med H2-database lokalt. For å kun konfigurere
                H2 når H2 er tilgjengelig på classpath og "spring.datasource.driverClassName" er
                satt til "org.h2.Driver", forsøker vi å laste klassen med Class.forName(..) og konfigurerer den
                ved hjelp av refleksjon dersom den eksisterer.

                På denne måten får vi konfigurert H2 når den er og skal være tilgjengelig, men ikke
                i produksjon der vi ikke bruker H2.

                Koden ligger i konstruktøren her for å sikre at den alltid blir kjørt.
             */
            log.info("Setter opp H2 med ekstra parameter for Oracle-kompatibilitet." )

            val modeClass = Class.forName("org.h2.engine.Mode")
            val getInstanceMethod = modeClass.getDeclaredMethod("getInstance", String::class.java)
            val modeInstance = getInstanceMethod.invoke(null, "ORACLE")
            val declaredField = modeInstance.javaClass.getDeclaredField("numericWithBooleanComparison")
            declaredField.set(modeInstance, true)
        }
    }

    @Bean
    @ConditionalOnProperty(name = ["ulykkesanalyse.miljo.fyll-database"], havingValue = "true")
    fun dataSource(): DataSource {
        return DataSourceBuilder.create()
            .driverClassName(dataSourceProperties.driverClassName)
            .url(dataSourceProperties.url)
            .username(dataSourceProperties.username)
            .password(dataSourceProperties.password)
            .build()
    }

    @Bean
    @ConditionalOnProperty(name = ["ulykkesanalyse.miljo.fyll-database"], havingValue = "true")
    fun dataSourceInitializer(): DataSourceInitializer {

        val databasePopulator = UlykkesanalyseDatabasePopulator()
        val kodeverkMangler =
            JdbcTemplate(dataSource()).queryForList("select ID from KODEVERK", String::class.java).size == 0
        if (kodeverkMangler) {
            databasePopulator.leggtilSkript(ClassPathResource("kodeverk.sql"))
            databasePopulator.leggtilSkript(ClassPathResource("autosysenhetskoder.sql"))
            databasePopulator.leggtilSkript(ClassPathResource("geografiskForhold.sql"))
            databasePopulator.leggtilSkript(ClassPathResource("handlingForRolle.sql"))
        }
        val dataSourceInitializer = DataSourceInitializer()
        dataSourceInitializer.setDataSource(dataSource())
        dataSourceInitializer.setDatabasePopulator(databasePopulator)
        dataSourceInitializer.setEnabled(true)
        return dataSourceInitializer;
    }
}
```


Abstract class to configure repository test:
```
@ExtendWith(SpringExtension::class)
@DataJpaTest
@Sql("/init-data.sql")
@ComponentScan("path.to.repository.classes")
@AutoConfigureTestDatabase(replace = AutoConfigureTestDatabase.Replace.NONE)
@ActiveProfiles("junit")
@Import(AbstractRepositoryTest.DS::class)
abstract class AbstractRepositoryTest(){
    @Autowired
    lateinit var entityManager: TestEntityManager

    @TestConfiguration
    internal class DS(){
        val logger = LoggerFactory.getLogger(this::class.java)
        @Bean
        fun h2DataSource(): DataSource {
            logger.info("Configuring H2 Oracle mode")
            val dbNameAndProperties =
                "${UUID.randomUUID()};Mode=Oracle;DB_CLOSE_DELAY=-1;" +
                "DB_CLOSE_ON_EXIT=false;DEFAULT_NULL_ORDERING=HIGH"

            val embeddedDatabase = EmbeddedDatabaseBuilder()
                .setType(EmbeddedDatabaseType.H2)
                .setName(dbNameAndProperties)
                .build()

            val mode = Mode.getInstance("ORACLE")
            mode.limit = true
            mode.numericWithBooleanComparison = true
            return embeddedDatabase
        }
    }
}
```
