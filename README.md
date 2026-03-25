# Java — Documentação Intermediária

## Índice

1. [Programação Orientada a Objetos (POO)](#1-programação-orientada-a-objetos-poo)
2. [Coleções e Estruturas de Dados](#2-coleções-e-estruturas-de-dados)
3. [Streams e Programação Funcional](#3-streams-e-programação-funcional)
4. [Tratamento de Exceções](#4-tratamento-de-exceções)
5. [Arquivos e I/O](#5-arquivos-e-io)
6. [Comandos e Ferramentas Úteis](#6-comandos-e-ferramentas-úteis)
7. [Design Patterns](#7-design-patterns)
8. [Boas Práticas](#8-boas-práticas)

---

## 1. Programação Orientada a Objetos (POO)

### Classe e objeto

```java
public class Produto {
    private String nome;
    private double preco;
    private int estoque;

    public Produto(String nome, double preco) {
        this.nome  = nome;
        this.preco = preco;
        this.estoque = 0;
    }

    public void adicionarEstoque(int qtd) {
        if (qtd > 0) this.estoque += qtd;
    }

    public boolean temEstoque() {
        return estoque > 0;
    }

    public String getNome()  { return nome; }
    public double getPreco() { return preco; }
    public int getEstoque()  { return estoque; }
}

Produto p = new Produto("Notebook", 3500.0);
p.adicionarEstoque(10);
System.out.println(p.temEstoque()); // true
```

### Os 4 Pilares

#### Encapsulamento

```java
private double saldo;

public void depositar(double valor) {
    if (valor > 0) saldo += valor;
}
```

#### Herança

```java
public class Animal {
    protected String nome;
    public Animal(String nome) { this.nome = nome; }
    public void dormir() { System.out.println(nome + " dormindo..."); }
}

public class Cachorro extends Animal {
    public Cachorro(String nome) { super(nome); }

    @Override
    public void fazerSom() { System.out.println("Au au!"); }
}
```

#### Polimorfismo

```java
Animal animal = new Cachorro("Rex");
animal.fazerSom(); // executa a versão do Cachorro

List<Animal> animais = List.of(new Cachorro("Rex"), new Gato("Mia"));
animais.forEach(Animal::fazerSom);
```

#### Abstração

```java
public interface Pagavel {
    void pagar(double valor);
    double getSaldo();
}

public abstract class Conta implements Pagavel {
    protected double saldo;

    public double getSaldo() { return saldo; }

    public abstract void calcularTaxa();
}
```

### Modificadores de acesso

| Modificador | Mesma classe | Mesmo pacote | Subclasse | Qualquer lugar |
| --- | --- | --- | --- | --- |
| `private` | Sim | Não | Não | Não |
| (default) | Sim | Sim | Não | Não |
| `protected` | Sim | Sim | Sim | Não |
| `public` | Sim | Sim | Sim | Sim |

### Interface vs Classe Abstrata

```java
public interface Voavel {
    void voar();
    default void pousar() { System.out.println("Pousando..."); }
}

public abstract class Ave {
    protected String nome;
    public Ave(String nome) { this.nome = nome; }
    public abstract void emitirSom();
    public void dormir() { System.out.println(nome + " dormindo..."); }
}

public class Aguia extends Ave implements Voavel {
    public Aguia() { super("Águia"); }
    public void emitirSom() { System.out.println("Grasnido!"); }
    public void voar()      { System.out.println("Voando alto!"); }
}
```

| | Interface | Classe Abstrata |
| --- | --- | --- |
| Herança múltipla | Sim | Não |
| Estado (atributos) | Não | Sim |
| Construtor | Não | Sim |
| Quando usar | Capacidade / contrato | Base comum com código compartilhado |

### Enum

```java
public enum Status {
    PENDENTE, PROCESSANDO, CONCLUIDO, CANCELADO;
}

public enum DiaSemana {
    SEGUNDA("Seg"), TERCA("Ter"), QUARTA("Qua");

    private final String abrev;
    DiaSemana(String abrev) { this.abrev = abrev; }
    public String getAbrev() { return abrev; }
}

switch (status) {
    case PENDENTE  -> System.out.println("Aguardando...");
    case CONCLUIDO -> System.out.println("Pronto!");
}
```

### Record (Java 16+)

```java
// Gera automaticamente: construtor, getters, equals, hashCode, toString
public record Produto(String nome, double preco, String categoria) {}

var p = new Produto("Notebook", 3500.0, "Eletrônico");
System.out.println(p.nome());  // "Notebook"
System.out.println(p.preco()); // 3500.0
```

### Generics

```java
public class Caixa<T> {
    private T conteudo;
    public void guardar(T item) { this.conteudo = item; }
    public T abrir() { return conteudo; }
}

Caixa<String> caixa = new Caixa<>();
caixa.guardar("surpresa");
System.out.println(caixa.abrir()); // "surpresa"
```

---

## 2. Coleções e Estruturas de Dados

### List — lista ordenada com duplicatas

```java
List<String> lista = new ArrayList<>();
lista.add("Ana");
lista.add("Carlos");
lista.add(0, "Beatriz");  // insere na posição

lista.get(0);             // acessar por índice
lista.remove("Ana");      // remover por valor
lista.remove(0);          // remover por índice
lista.contains("Carlos"); // verificar existência
lista.size();
lista.isEmpty();
Collections.sort(lista);
```

### Map — chave e valor

```java
Map<String, Integer> mapa = new HashMap<>();
mapa.put("maçã", 3);
mapa.put("banana", 5);

mapa.get("maçã");                // 3
mapa.getOrDefault("laranja", 0); // 0 se não existir
mapa.containsKey("banana");      // true
mapa.remove("maçã");

mapa.forEach((chave, valor) ->
    System.out.println(chave + " → " + valor));
```

### Set — sem duplicatas

```java
Set<String> conjunto = new HashSet<>();
conjunto.add("Java");
conjunto.add("Java"); // ignorado — duplicata
conjunto.contains("Java"); // true
conjunto.size(); // 1
```

### Queue — fila FIFO

```java
Queue<String> fila = new LinkedList<>();
fila.offer("primeiro");
fila.offer("segundo");
fila.peek(); // ver sem remover → "primeiro"
fila.poll(); // remove e retorna → "primeiro"
```

### Deque — pilha LIFO

```java
Deque<String> pilha = new ArrayDeque<>();
pilha.push("base");
pilha.push("topo");
pilha.peek(); // ver o topo → "topo"
pilha.pop();  // remove o topo → "topo"
```

---

## 3. Streams e Programação Funcional

### Operações principais

```java
List<Integer> numeros = List.of(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);

// filter
List<Integer> pares = numeros.stream()
    .filter(n -> n % 2 == 0)
    .collect(Collectors.toList());

// map
List<String> textos = numeros.stream()
    .map(n -> "Nº " + n)
    .collect(Collectors.toList());

// sorted
List<Integer> desc = numeros.stream()
    .sorted((a, b) -> b - a)
    .collect(Collectors.toList());

// agregar
int soma = numeros.stream().mapToInt(Integer::intValue).sum();
int max  = numeros.stream().mapToInt(Integer::intValue).max().getAsInt();

// findFirst
Optional<Integer> primeiro = numeros.stream()
    .filter(n -> n > 5)
    .findFirst();

// anyMatch / allMatch / noneMatch
boolean algumPar = numeros.stream().anyMatch(n -> n % 2 == 0); // true
boolean todosPar = numeros.stream().allMatch(n -> n % 2 == 0); // false
```

### Functional Interfaces

```java
import java.util.function.*;

Function<String, Integer>  tamanho  = String::length;      // recebe T, retorna R
Predicate<Integer>         ehPar    = n -> n % 2 == 0;     // retorna boolean
Consumer<String>           imprimir = System.out::println;  // não retorna
Supplier<String>           saudacao = () -> "Olá!";         // não recebe
```

### Optional — evitar NullPointerException

```java
Optional<String> nome = Optional.ofNullable(buscarNome());

nome.isPresent();                    // tem valor?
nome.get();                          // obtém (lança exceção se vazio)
nome.orElse("Desconhecido");         // valor padrão
nome.orElseGet(() -> gerarNome());   // valor padrão lazy
nome.ifPresent(System.out::println); // executa se presente
```

---

## 4. Tratamento de Exceções

### Tipos e uso

```java
// Checked — obriga try/catch (IOException, SQLException...)
// Unchecked — opcional (RuntimeException, NullPointerException...)

try {
    int resultado = dividir(10, 0);
} catch (ArithmeticException e) {
    System.out.println("Erro: " + e.getMessage());
} finally {
    System.out.println("Sempre executa");
}

// Multi-catch
try {
    processar();
} catch (IOException | SQLException e) {
    System.out.println("Erro: " + e.getMessage());
}

// try-with-resources — fecha automaticamente
try (BufferedReader reader = Files.newBufferedReader(Path.of("arquivo.txt"))) {
    String linha = reader.readLine();
} catch (IOException e) {
    e.printStackTrace();
}
```

### Exceção customizada

```java
public class SaldoInsuficienteException extends RuntimeException {
    private final double saldoAtual;
    private final double valorSolicitado;

    public SaldoInsuficienteException(double saldoAtual, double valorSolicitado) {
        super(String.format("Saldo insuficiente. Saldo: %.2f | Solicitado: %.2f",
              saldoAtual, valorSolicitado));
        this.saldoAtual      = saldoAtual;
        this.valorSolicitado = valorSolicitado;
    }
}

if (valor > saldo) throw new SaldoInsuficienteException(saldo, valor);
```

---

## 5. Arquivos e I/O

### Operações básicas com NIO

```java
import java.nio.file.*;

Path arquivo = Path.of("dados/arquivo.txt");

// Ler
String conteudo     = Files.readString(arquivo);
List<String> linhas = Files.readAllLines(arquivo, StandardCharsets.UTF_8);

// Escrever
Files.writeString(arquivo, "conteúdo");
Files.write(arquivo, List.of("linha1", "linha2"), StandardCharsets.UTF_8);

// Gerenciar
Files.exists(arquivo);
Files.copy(arquivo, Path.of("backup/arquivo.txt"));
Files.move(arquivo, Path.of("processados/arquivo.txt"));
Files.delete(arquivo);
Files.createDirectories(Path.of("pasta/subpasta"));
```

### Ler arquivo grande (linha a linha)

```java
try (BufferedReader reader = Files.newBufferedReader(Path.of("grande.txt"))) {
    String linha;
    while ((linha = reader.readLine()) != null) {
        processar(linha);
    }
}
```

### Listar arquivos de uma pasta

```java
// Pasta atual
Files.list(Path.of("importacoes"))
     .filter(p -> p.toString().endsWith(".csv"))
     .forEach(System.out::println);

// Recursivo
Files.walk(Path.of("importacoes"))
     .filter(Files::isRegularFile)
     .forEach(System.out::println);
```

---

## 6. Comandos e Ferramentas Úteis

### Compilar e executar

```bash
javac MeuPrograma.java           # compilar
java MeuPrograma                 # executar
javac -cp "libs/*" src/Main.java -d out/  # com dependências
java -cp "out:libs/*" Main
java -version
```

### Maven

```bash
mvn archetype:generate -DgroupId=com.exemplo -DartifactId=meu-projeto
mvn compile          # compilar
mvn test             # testes
mvn package          # gerar JAR
mvn clean            # limpar
mvn clean package    # compilar + testar + gerar JAR
mvn install          # instalar no repositório local
mvn dependency:resolve
```

### Gradle

```bash
./gradlew build      # compilar
./gradlew test       # testes
./gradlew clean      # limpar
./gradlew run        # executar
```

### JAR

```bash
jar cfe app.jar com.exemplo.Main -C out/ .  # criar JAR executável
java -jar app.jar                           # executar
jar tf app.jar                              # listar conteúdo
```

### JVM — opções úteis

```bash
java -Xms256m -Xmx1g -jar app.jar           # definir memória heap
java -Dambiente=producao -jar app.jar        # variável de sistema
```

```java
String env = System.getProperty("ambiente"); // ler propriedade
String env = System.getenv("APP_ENV");       // ler variável de ambiente
```

### Datas (Java 8+)

```java
LocalDate hoje   = LocalDate.now();
LocalTime agora  = LocalTime.now();
LocalDateTime dt = LocalDateTime.now();

DateTimeFormatter fmt = DateTimeFormatter.ofPattern("dd/MM/yyyy");
String texto   = hoje.format(fmt);
LocalDate data = LocalDate.parse("25/03/2026", fmt);

hoje.plusDays(1);
hoje.minusWeeks(1);
ChronoUnit.DAYS.between(data1, data2);

data1.isBefore(data2);
data1.isAfter(data2);
data1.isEqual(data2);
```

---

## 7. Design Patterns

### Criacionais

| Pattern | Quando usar |
| --- | --- |
| **Singleton** | Uma única instância (config, log, conexão) |
| **Factory** | Decidir qual classe instanciar |
| **Builder** | Objetos complexos com muitos parâmetros |

```java
// Singleton
public class Config {
    private static Config instancia;
    private Config() {}
    public static Config getInstance() {
        if (instancia == null) instancia = new Config();
        return instancia;
    }
}

// Builder
Pedido pedido = new Pedido.Builder()
    .produto("Notebook")
    .quantidade(1)
    .expresso()
    .build();

// Factory
Notificacao n = NotificacaoFactory.criar("email");
n.enviar("Olá!");
```

### Estruturais

| Pattern | Quando usar |
| --- | --- |
| **Adapter** | Integrar interfaces incompatíveis |
| **Decorator** | Adicionar comportamento sem herança |

### Comportamentais

| Pattern | Quando usar |
| --- | --- |
| **Strategy** | Trocar algoritmo em tempo de execução |
| **Observer** | Notificar múltiplos objetos sobre eventos |
| **Command** | Encapsular ações com suporte a undo/redo |

---

## 8. Boas Práticas

### Nomenclatura

```java
class ContaBancaria { }              // PascalCase
void calcularSaldo() { }             // camelCase
double valorTotal = 0;               // camelCase
static final double TAXA_JUROS = 0.02; // SNAKE_UPPER_CASE
package com.empresa.financeiro;      // minúsculas
```

### Princípios SOLID

| Princípio | Resumo |
| --- | --- |
| **S** — Single Responsibility | Uma classe, uma responsabilidade |
| **O** — Open/Closed | Aberta para extensão, fechada para modificação |
| **L** — Liskov Substitution | Subclasses devem substituir a classe pai |
| **I** — Interface Segregation | Interfaces pequenas e específicas |
| **D** — Dependency Inversion | Dependa de abstrações, não de implementações |

### Checklist de código limpo

```text
✔ Nomes descritivos para variáveis, métodos e classes
✔ Métodos pequenos — fazem apenas uma coisa
✔ Sem números mágicos — use constantes nomeadas
✔ Tratar exceções com mensagens úteis
✔ Evitar null — use Optional
✔ Fechar recursos com try-with-resources
✔ Escrever testes para o código crítico
✔ Nunca expor senhas ou chaves no código — usar variáveis de ambiente
```

### Estrutura de pacotes recomendada

```text
src/main/java/com/empresa/
├── controller/     # entrada HTTP (APIs)
├── service/        # regras de negócio
├── repository/     # acesso a dados
├── model/          # entidades e records
├── dto/            # objetos de transferência
├── exception/      # exceções customizadas
└── config/         # configurações
```

---

## Documentações Específicas

| Documento | Conteúdo |
| --- | --- |
| [java-guia-completo.md](../java-guia-completo.md) | OOP, Streams e Design Patterns com exemplos |
| [java-trilha-estudos.md](../java-trilha-estudos.md) | Roadmap e checklist de estudos |
| [java-arquivos-seguranca.md](../java-arquivos-seguranca.md) | Arquivos, importação e criptografia |
