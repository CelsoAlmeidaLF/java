# Trilha de Estudos Java — Próximos Passos

## Índice

1. [Fundamentos que Faltam](#1-fundamentos-que-faltam)
2. [Intermediário](#2-intermediário)
3. [Build e Projeto Real](#3-build-e-projeto-real)
4. [Web e Backend](#4-web-e-backend)
5. [Ordem de Estudo Recomendada](#5-ordem-de-estudo-recomendada)

---

## 1. Fundamentos que Faltam

### 1.1 Modificadores de Acesso

Controlam a visibilidade de classes, atributos e métodos.

```java
public class Exemplo {
    public    String visivelParaTodos;     // qualquer classe
    protected String visivelNoPacoteEHeranca; // mesmo pacote + subclasses
    String            visivelNoPacote;     // apenas o mesmo pacote (default)
    private   String  visivelSoAqui;      // apenas esta classe
}
```

| Modificador | Mesma classe | Mesmo pacote | Subclasse | Qualquer lugar |
|---|---|---|---|---|
| `private` | Sim | Não | Não | Não |
| default | Sim | Sim | Não | Não |
| `protected` | Sim | Sim | Sim | Não |
| `public` | Sim | Sim | Sim | Sim |

---

### 1.2 Interface vs Classe Abstrata

```java
// Interface — contrato puro, sem estado
public interface Voavel {
    void voar(); // obrigatório implementar

    default void pousar() {
        System.out.println("Pousando..."); // comportamento padrão (Java 8+)
    }
}

// Classe abstrata — base com estado e comportamento parcial
public abstract class Ave {
    protected String nome;

    public Ave(String nome) { this.nome = nome; }

    public abstract void emitirSom(); // obrigatório implementar

    public void dormir() { // já implementado
        System.out.println(nome + " dormindo...");
    }
}

// Combinando os dois
public class Aguia extends Ave implements Voavel {
    public Aguia() { super("Águia"); }

    public void emitirSom() { System.out.println("Grasnido!"); }
    public void voar()      { System.out.println("Voando alto!"); }
}
```

| | Interface | Classe Abstrata |
|---|---|---|
| Herança múltipla | Sim (`implements A, B`) | Não (só uma) |
| Estado (atributos) | Não | Sim |
| Construtor | Não | Sim |
| Quando usar | Contrato / capacidade | Base comum com código compartilhado |

---

### 1.3 Enums

Tipos com valores fixos e seguros.

```java
public enum Status {
    PENDENTE, PROCESSANDO, CONCLUIDO, CANCELADO;
}

public enum DiaSemana {
    SEGUNDA("Seg"), TERCA("Ter"), QUARTA("Qua"),
    QUINTA("Qui"), SEXTA("Sex"), SABADO("Sáb"), DOMINGO("Dom");

    private final String abreviacao;

    DiaSemana(String abreviacao) { this.abreviacao = abreviacao; }

    public String getAbreviacao() { return abreviacao; }
}

// Uso
Status status = Status.PENDENTE;

switch (status) {
    case PENDENTE    -> System.out.println("Aguardando...");
    case CONCLUIDO   -> System.out.println("Pronto!");
    case CANCELADO   -> System.out.println("Cancelado.");
}

System.out.println(DiaSemana.SEGUNDA.getAbreviacao()); // "Seg"
```

---

### 1.4 Annotations

```java
public class Produto {

    @Deprecated // sinaliza que o método será removido
    public void calcularPrecoAntigo() { }

    @Override // garante que está sobrescrevendo corretamente
    public String toString() {
        return "Produto{}";
    }

    @SuppressWarnings("unchecked") // suprime aviso do compilador
    public void metodo() { }
}
```

---

### 1.5 Tratamento de Exceções Avançado

```java
// Checked vs Unchecked
// Checked — obriga try/catch (ex: IOException, SQLException)
// Unchecked — opcional (ex: RuntimeException, NullPointerException)

// try-with-resources — fecha o recurso automaticamente
try (BufferedReader reader = new BufferedReader(new FileReader("arquivo.txt"))) {
    String linha = reader.readLine();
} catch (IOException e) {
    e.printStackTrace();
} // reader.close() chamado automaticamente

// Multi-catch
try {
    processar();
} catch (IOException | SQLException e) {
    System.out.println("Erro de I/O ou banco: " + e.getMessage());
}

// Relançar com contexto
try {
    conectar();
} catch (SQLException e) {
    throw new RuntimeException("Falha ao conectar ao banco", e);
}
```

---

## 2. Intermediário

### 2.1 Functional Interfaces

Interfaces com um único método abstrato — usadas com lambdas.

```java
import java.util.function.*;

// Function<T, R> — recebe T, retorna R
Function<String, Integer> tamanho = String::length;
tamanho.apply("Java"); // 4

// Predicate<T> — recebe T, retorna boolean
Predicate<Integer> ehPar = n -> n % 2 == 0;
ehPar.test(4); // true

// Consumer<T> — recebe T, não retorna nada
Consumer<String> imprimir = System.out::println;
imprimir.accept("Olá!");

// Supplier<T> — não recebe nada, retorna T
Supplier<String> saudacao = () -> "Bem-vindo!";
saudacao.get(); // "Bem-vindo!"

// BiFunction<T, U, R> — recebe dois argumentos
BiFunction<String, Integer, String> repetir = String::repeat;
repetir.apply("ab", 3); // "ababab"
```

---

### 2.2 Lambda e Method Reference

```java
List<String> nomes = List.of("Carlos", "Ana", "Beatriz");

// Lambda
nomes.stream()
     .filter(n -> n.startsWith("A"))
     .forEach(n -> System.out.println(n));

// Method Reference — forma mais enxuta
nomes.stream()
     .filter(n -> n.startsWith("A"))
     .forEach(System.out::println); // equivalente ao lambda acima

// Tipos de Method Reference
NomeDaClasse::metodoEstatico   // Math::abs
objeto::metodoDeInstancia      // System.out::println
NomeDaClasse::metodoDeInstancia // String::toUpperCase
NomeDaClasse::new              // Produto::new (constructor reference)
```

---

### 2.3 Generics Avançado

```java
// Wildcard — aceita qualquer tipo
public void imprimirLista(List<?> lista) {
    lista.forEach(System.out::println);
}

// Upper bounded — aceita T e subtipos de T
public double somar(List<? extends Number> numeros) {
    return numeros.stream().mapToDouble(Number::doubleValue).sum();
}

// Lower bounded — aceita T e supertipos de T
public void adicionarNumeros(List<? super Integer> lista) {
    lista.add(1);
    lista.add(2);
}
```

---

### 2.4 Concorrência

```java
// CompletableFuture — operações assíncronas
CompletableFuture<String> future = CompletableFuture
    .supplyAsync(() -> buscarDados())       // roda em outra thread
    .thenApply(dados -> processar(dados))   // transforma o resultado
    .thenApply(String::toUpperCase);

String resultado = future.get(); // aguarda o resultado

// Encadear múltiplas tarefas em paralelo
CompletableFuture<String> f1 = CompletableFuture.supplyAsync(() -> "Dados A");
CompletableFuture<String> f2 = CompletableFuture.supplyAsync(() -> "Dados B");

CompletableFuture.allOf(f1, f2).join(); // aguarda ambas
```

---

### 2.5 I/O e Arquivos

```java
import java.nio.file.*;

Path arquivo = Path.of("dados.txt");

// Ler
String conteudo = Files.readString(arquivo);
List<String> linhas = Files.readAllLines(arquivo);

// Escrever
Files.writeString(arquivo, "conteúdo");
Files.write(arquivo, List.of("linha1", "linha2"));

// Verificar e criar
Files.exists(arquivo);
Files.createDirectories(Path.of("pasta/subpasta"));

// Listar arquivos de um diretório
Files.list(Path.of("."))
     .filter(p -> p.toString().endsWith(".txt"))
     .forEach(System.out::println);
```

---

## 3. Build e Projeto Real

### 3.1 Maven — estrutura e dependências

```
meu-projeto/
├── pom.xml                 # configuração do projeto
└── src/
    ├── main/
    │   └── java/           # código principal
    └── test/
        └── java/           # testes
```

```xml
<!-- pom.xml -->
<dependencies>
    <!-- Spring Boot -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
        <version>3.2.0</version>
    </dependency>

    <!-- Testes -->
    <dependency>
        <groupId>org.junit.jupiter</groupId>
        <artifactId>junit-jupiter</artifactId>
        <version>5.10.0</version>
        <scope>test</scope>
    </dependency>
</dependencies>
```

---

### 3.2 JUnit 5 — testes unitários

```java
import org.junit.jupiter.api.*;
import static org.junit.jupiter.api.Assertions.*;

class ContaBancariaTest {

    private ContaBancaria conta;

    @BeforeEach
    void setUp() {
        conta = new ContaBancaria();
        conta.depositar(1000.0);
    }

    @Test
    void deveDepositarCorretamente() {
        conta.depositar(500.0);
        assertEquals(1500.0, conta.getSaldo());
    }

    @Test
    void deveLancarExcecaoAoSacarMaisQueOSaldo() {
        assertThrows(SaldoInsuficienteException.class, () -> {
            conta.sacar(2000.0);
        });
    }

    @Test
    void naoDeveDepositarValorNegativo() {
        conta.depositar(-100.0);
        assertEquals(1000.0, conta.getSaldo()); // saldo não muda
    }
}
```

---

### 3.3 Mockito — simular dependências

```java
import static org.mockito.Mockito.*;

@ExtendWith(MockitoExtension.class)
class PedidoServiceTest {

    @Mock
    private PedidoRepository repositorio; // simula o banco de dados

    @InjectMocks
    private PedidoService service;

    @Test
    void deveBuscarPedidoPorId() {
        Pedido pedidoFake = new Pedido("Notebook", 1);
        when(repositorio.findById(1L)).thenReturn(Optional.of(pedidoFake));

        Pedido resultado = service.buscar(1L);

        assertEquals("Notebook", resultado.getProduto());
        verify(repositorio).findById(1L); // confirma que foi chamado
    }
}
```

---

### 3.4 Logging com SLF4J

```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class PedidoService {
    private static final Logger log = LoggerFactory.getLogger(PedidoService.class);

    public void processar(Pedido pedido) {
        log.info("Processando pedido: {}", pedido.getId());

        try {
            // lógica
            log.debug("Pedido processado com sucesso");
        } catch (Exception e) {
            log.error("Erro ao processar pedido {}: {}", pedido.getId(), e.getMessage());
        }
    }
}
```

---

## 4. Web e Backend

### 4.1 Spring Boot — estrutura de uma API REST

```
src/main/java/com/exemplo/
├── controller/     # recebe as requisições HTTP
├── service/        # regras de negócio
├── repository/     # acesso ao banco de dados
├── model/          # entidades
└── Application.java
```

```java
// Controller
@RestController
@RequestMapping("/produtos")
public class ProdutoController {

    @Autowired
    private ProdutoService service;

    @GetMapping
    public List<Produto> listar() {
        return service.listarTodos();
    }

    @GetMapping("/{id}")
    public Produto buscar(@PathVariable Long id) {
        return service.buscar(id);
    }

    @PostMapping
    @ResponseStatus(HttpStatus.CREATED)
    public Produto criar(@RequestBody @Valid Produto produto) {
        return service.salvar(produto);
    }

    @DeleteMapping("/{id}")
    @ResponseStatus(HttpStatus.NO_CONTENT)
    public void deletar(@PathVariable Long id) {
        service.deletar(id);
    }
}
```

---

### 4.2 JPA + Hibernate — banco de dados

```java
// Entidade mapeada para tabela
@Entity
@Table(name = "produtos")
public class Produto {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false)
    private String nome;

    private double preco;
}

// Repository — operações no banco sem SQL manual
public interface ProdutoRepository extends JpaRepository<Produto, Long> {

    List<Produto> findByCategoria(String categoria);

    @Query("SELECT p FROM Produto p WHERE p.preco < :valor")
    List<Produto> buscarBaratos(@Param("valor") double valor);
}
```

---

### 4.3 Validação com Bean Validation

```java
public class Produto {

    @NotBlank(message = "Nome é obrigatório")
    private String nome;

    @Positive(message = "Preço deve ser positivo")
    private double preco;

    @NotNull
    @Size(min = 3, max = 50)
    private String categoria;

    @Email
    private String emailFornecedor;
}
```

---

### 4.4 Docker — empacotar a aplicação

```dockerfile
# Dockerfile
FROM eclipse-temurin:21-jdk-alpine
WORKDIR /app
COPY target/minha-app.jar app.jar
ENTRYPOINT ["java", "-jar", "app.jar"]
```

```yaml
# docker-compose.yml
services:
  app:
    build: .
    ports:
      - "8080:8080"
    environment:
      - SPRING_DATASOURCE_URL=jdbc:postgresql://db:5432/meubanco

  db:
    image: postgres:16
    environment:
      POSTGRES_DB: meubanco
      POSTGRES_PASSWORD: senha
```

---

## 5. Ordem de Estudo Recomendada

```
FASE 1 — Completar os Fundamentos
    → Modificadores de acesso
    → Interface vs Classe Abstrata
    → Enums
    → Tratamento de exceções avançado

FASE 2 — Programação Funcional
    → Functional Interfaces (Function, Predicate, Consumer, Supplier)
    → Lambda e Method Reference
    → Generics avançado

FASE 3 — Projeto Real
    → Maven (estrutura e dependências)
    → JUnit 5 (testes unitários)
    → Mockito (mocks)
    → Logging com SLF4J

FASE 4 — Backend com Spring Boot
    → Spring Boot básico (Controller, Service, Repository)
    → JPA + Hibernate (banco de dados)
    → Validação (Bean Validation)
    → Spring Security (autenticação e autorização)

FASE 5 — Produção
    → Docker (empacotar a aplicação)
    → Variáveis de ambiente por perfil (dev/prod)
    → CI/CD básico (GitHub Actions)
```

### Checklist de progresso

- [x] Orientação a Objetos
- [x] Listas, Filas e Pilhas
- [x] Streams (equivalente ao LINQ)
- [x] Design Patterns
- [ ] Modificadores de acesso
- [ ] Interface vs Classe Abstrata avançado
- [ ] Enums
- [ ] Exceções avançadas
- [ ] Functional Interfaces e Lambda
- [ ] Generics avançado
- [ ] Maven
- [ ] JUnit 5 + Mockito
- [ ] Logging
- [ ] Spring Boot
- [ ] JPA + Hibernate
- [ ] Docker
