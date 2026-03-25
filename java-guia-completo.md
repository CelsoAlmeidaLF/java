# Guia Completo de Java

## Índice

1. [Orientação a Objetos](#1-orientação-a-objetos)
2. [Fundamentos Essenciais](#2-fundamentos-essenciais)
3. [Listas, Filas, Pilhas e Streams](#3-listas-filas-pilhas-e-streams)
4. [Design Patterns](#4-design-patterns)

---

## 1. Orientação a Objetos

Os 4 pilares da OOP em Java:

### 1.1 Encapsulamento

Protege o estado interno da classe expondo apenas o necessário.

```java
public class ContaBancaria {
    private double saldo;

    public double getSaldo() {
        return saldo;
    }

    public void depositar(double valor) {
        if (valor > 0) saldo += valor;
    }
}
```

### 1.2 Herança

Permite que uma classe herde comportamento de outra.

```java
public class Animal {
    public void fazerSom() {
        System.out.println("Som genérico");
    }
}

public class Cachorro extends Animal {
    @Override
    public void fazerSom() {
        System.out.println("Au au!");
    }
}
```

### 1.3 Polimorfismo

Um mesmo tipo pode assumir diferentes formas em tempo de execução.

```java
Animal animal = new Cachorro();
animal.fazerSom(); // "Au au!"

List<Animal> animais = List.of(new Cachorro(), new Gato());
animais.forEach(Animal::fazerSom);
```

### 1.4 Abstração

Define contratos e comportamentos base sem expor detalhes de implementação.

```java
// Interface — contrato puro
public interface Pagamento {
    void processar(double valor);
}

// Classe abstrata — base parcial com comportamento comum
public abstract class Veiculo {
    protected String marca;

    public abstract void acelerar();

    public void frear() {
        System.out.println("Freando...");
    }
}
```

### Quando usar cada recurso

| Situação | Use |
|---|---|
| Compartilhar comportamento entre classes | `abstract class` |
| Definir um contrato sem implementação | `interface` |
| Reutilizar código com variação | herança + `@Override` |
| Proteger estado interno | `private` + getters/setters |

---

## 2. Fundamentos Essenciais

### 2.1 Tipos e variáveis

```java
int idade = 25;
double preco = 9.99;
boolean ativo = true;
String nome = "João";          // String é objeto, não primitivo
var lista = new ArrayList<>(); // inferência de tipo (Java 10+)
```

### 2.2 Coleções

```java
List<String> lista = new ArrayList<>();
Map<String, Integer> mapa = new HashMap<>();
Set<String> conjunto = new HashSet<>();

lista.add("item");
mapa.put("chave", 42);
mapa.getOrDefault("chave", 0);
```

### 2.3 Records (Java 16+)

Gera automaticamente construtor, getters, `equals`, `hashCode` e `toString`.

```java
public record Produto(String nome, double preco) {}

var p = new Produto("Cadeira", 299.90);
System.out.println(p.nome());
```

### 2.4 Optional — evitar NullPointerException

```java
Optional<String> nome = Optional.ofNullable(buscarNome());
String resultado = nome.orElse("Desconhecido");
```

### 2.5 Tratamento de erros

```java
try {
    int resultado = dividir(10, 0);
} catch (ArithmeticException e) {
    System.out.println("Erro: " + e.getMessage());
} finally {
    System.out.println("Sempre executa");
}

// Exceção customizada
public class SaldoInsuficienteException extends RuntimeException {
    public SaldoInsuficienteException(String msg) { super(msg); }
}
```

### 2.6 Generics

```java
public class Caixa<T> {
    private T conteudo;

    public void guardar(T item) { this.conteudo = item; }
    public T abrir() { return conteudo; }
}

Caixa<String> caixa = new Caixa<>();
caixa.guardar("surpresa");
```

### 2.7 Concorrência básica

```java
// Thread simples
new Thread(() -> System.out.println("Rodando em paralelo")).start();

// ExecutorService (preferível)
ExecutorService executor = Executors.newFixedThreadPool(4);
executor.submit(() -> processarTarefa());
executor.shutdown();
```

### Ecossistema Java

| Área | Ferramenta |
|---|---|
| Build / dependências | Maven ou Gradle |
| Framework web | Spring Boot |
| Banco de dados | JPA / Hibernate |
| Testes | JUnit 5 + Mockito |
| API REST | Spring MVC ou Quarkus |
| Containers | Docker + Spring Boot |

### Ordem de aprendizado sugerida

```
Sintaxe básica
    → OOP
        → Coleções + Streams
            → Exceções + Generics
                → Maven + Spring Boot
                    → JPA + banco de dados
                        → Testes automatizados
```

---

## 3. Listas, Filas, Pilhas e Streams

### 3.1 Lista — `List`

```java
List<String> nomes = new ArrayList<>();

nomes.add("Ana");
nomes.add("Carlos");
nomes.add(0, "Beatriz"); // insere na posição

String primeiro = nomes.get(0);

nomes.remove("Ana");
nomes.remove(0);        // por índice

nomes.contains("Carlos"); // true
nomes.size();
nomes.isEmpty();
```

### 3.2 Fila — `Queue` (FIFO)

Primeiro a entrar, primeiro a sair.

```java
Queue<String> fila = new LinkedList<>();

fila.offer("cliente1");
fila.offer("cliente2");
fila.offer("cliente3");

fila.peek(); // ver o primeiro sem remover → "cliente1"
fila.poll(); // remove e retorna o primeiro  → "cliente1"
```

### 3.3 Pilha — `Deque` (LIFO)

Último a entrar, primeiro a sair.

```java
Deque<String> pilha = new ArrayDeque<>();

pilha.push("base");
pilha.push("meio");
pilha.push("topo");

pilha.peek(); // ver o topo sem remover → "topo"
pilha.pop();  // remove o topo          → "topo"
```

### 3.4 Streams — equivalente ao LINQ

#### Comparação LINQ (C#) vs Streams (Java)

| LINQ (C#) | Streams (Java) |
|---|---|
| `.Where()` | `.filter()` |
| `.Select()` | `.map()` |
| `.OrderBy()` | `.sorted()` |
| `.FirstOrDefault()` | `.findFirst()` |
| `.Sum()` | `.mapToInt().sum()` |
| `.ToList()` | `.collect(Collectors.toList())` |
| `.Any()` | `.anyMatch()` |
| `.All()` | `.allMatch()` |
| `.Count()` | `.count()` |

#### Operações básicas

```java
List<Integer> numeros = List.of(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);

// Filtrar pares
List<Integer> pares = numeros.stream()
    .filter(n -> n % 2 == 0)
    .collect(Collectors.toList());

// Transformar
List<String> textos = numeros.stream()
    .map(n -> "Número: " + n)
    .collect(Collectors.toList());

// Ordenar decrescente
List<Integer> ordenado = numeros.stream()
    .sorted((a, b) -> b - a)
    .collect(Collectors.toList());

// Somar
int soma = numeros.stream()
    .mapToInt(Integer::intValue)
    .sum();

// Primeiro maior que 5
Optional<Integer> primeiro = numeros.stream()
    .filter(n -> n > 5)
    .findFirst();

// Verificar condições
boolean todosPares = numeros.stream().allMatch(n -> n % 2 == 0);
boolean algumPar   = numeros.stream().anyMatch(n -> n % 2 == 0);
```

#### Exemplo com objetos

```java
record Produto(String nome, double preco, String categoria) {}

List<Produto> produtos = List.of(
    new Produto("Notebook", 3500.0, "Eletrônico"),
    new Produto("Mouse",      80.0, "Eletrônico"),
    new Produto("Cadeira",   900.0, "Móvel"),
    new Produto("Mesa",     1200.0, "Móvel")
);

// Filtrar e ordenar por preço
List<Produto> resultado = produtos.stream()
    .filter(p -> p.categoria().equals("Eletrônico"))
    .filter(p -> p.preco() > 100)
    .sorted((a, b) -> Double.compare(a.preco(), b.preco()))
    .collect(Collectors.toList());

// Somar preços de uma categoria
double totalMoveis = produtos.stream()
    .filter(p -> p.categoria().equals("Móvel"))
    .mapToDouble(Produto::preco)
    .sum(); // 2100.0

// Agrupar por categoria
Map<String, List<Produto>> porCategoria = produtos.stream()
    .collect(Collectors.groupingBy(Produto::categoria));
```

### Quando usar cada estrutura

| Estrutura | Quando usar |
|---|---|
| `ArrayList` | Lista de acesso por índice, uso geral |
| `LinkedList` | Muitas inserções/remoções no meio |
| `Queue` | Fila de processamento (atendimento, tarefas) |
| `Deque` / pilha | Histórico, undo/redo, parsing |
| Streams | Filtrar, transformar e agregar coleções |

---

## 4. Design Patterns

Padrões divididos em 3 categorias (GoF — Gang of Four):

### 4.1 Criacionais — como criar objetos

#### Singleton — uma única instância global

Útil para configurações, conexões, logs.

```java
public class Configuracao {
    private static Configuracao instancia;
    private String ambiente;

    private Configuracao() { this.ambiente = "producao"; }

    public static Configuracao getInstance() {
        if (instancia == null) {
            instancia = new Configuracao();
        }
        return instancia;
    }
}

// Uso
Configuracao config = Configuracao.getInstance();
```

#### Factory Method — delegar a criação para subclasses

```java
public interface Notificacao {
    void enviar(String mensagem);
}

public class Email implements Notificacao {
    public void enviar(String msg) { System.out.println("Email: " + msg); }
}

public class SMS implements Notificacao {
    public void enviar(String msg) { System.out.println("SMS: " + msg); }
}

public class NotificacaoFactory {
    public static Notificacao criar(String tipo) {
        return switch (tipo) {
            case "email" -> new Email();
            case "sms"   -> new SMS();
            default -> throw new IllegalArgumentException("Tipo inválido");
        };
    }
}

// Uso
NotificacaoFactory.criar("email").enviar("Olá!");
```

#### Builder — construir objetos complexos passo a passo

```java
public class Pedido {
    private String produto;
    private int quantidade;
    private String endereco;
    private boolean expresso;

    private Pedido() {}

    public static class Builder {
        private Pedido pedido = new Pedido();

        public Builder produto(String v)   { pedido.produto = v; return this; }
        public Builder quantidade(int v)   { pedido.quantidade = v; return this; }
        public Builder endereco(String v)  { pedido.endereco = v; return this; }
        public Builder expresso()          { pedido.expresso = true; return this; }
        public Pedido build()              { return pedido; }
    }
}

// Uso
Pedido pedido = new Pedido.Builder()
    .produto("Notebook")
    .quantidade(1)
    .endereco("Rua A, 123")
    .expresso()
    .build();
```

---

### 4.2 Estruturais — como organizar classes

#### Adapter — compatibilizar interfaces incompatíveis

```java
// Sistema legado
public class PagamentoLegado {
    public void pagarEmReais(double valor) {
        System.out.println("Pagando R$" + valor);
    }
}

// Interface esperada pelo sistema atual
public interface Pagamento {
    void pagar(double valor);
}

// Adapter — ponte entre os dois
public class PagamentoAdapter implements Pagamento {
    private PagamentoLegado legado = new PagamentoLegado();

    public void pagar(double valor) {
        legado.pagarEmReais(valor);
    }
}
```

#### Decorator — adicionar comportamento dinamicamente

```java
public interface Cafe {
    String descricao();
    double preco();
}

public class CafeSimples implements Cafe {
    public String descricao() { return "Café"; }
    public double preco()     { return 2.0; }
}

public class ComLeite implements Cafe {
    private Cafe cafe;
    public ComLeite(Cafe cafe) { this.cafe = cafe; }

    public String descricao() { return cafe.descricao() + " + Leite"; }
    public double preco()     { return cafe.preco() + 1.0; }
}

public class ComChantilly implements Cafe {
    private Cafe cafe;
    public ComChantilly(Cafe cafe) { this.cafe = cafe; }

    public String descricao() { return cafe.descricao() + " + Chantilly"; }
    public double preco()     { return cafe.preco() + 1.5; }
}

// Uso — composição em camadas
Cafe pedido = new ComChantilly(new ComLeite(new CafeSimples()));
System.out.println(pedido.descricao()); // Café + Leite + Chantilly
System.out.println(pedido.preco());     // 4.5
```

---

### 4.3 Comportamentais — como os objetos se comunicam

#### Strategy — trocar algoritmo em tempo de execução

```java
public interface CalculoFrete {
    double calcular(double peso);
}

public class Sedex implements CalculoFrete {
    public double calcular(double peso) { return peso * 5.0; }
}

public class PAC implements CalculoFrete {
    public double calcular(double peso) { return peso * 2.0; }
}

public class Pedido {
    private CalculoFrete estrategia;

    public void setFrete(CalculoFrete estrategia) {
        this.estrategia = estrategia;
    }

    public double calcularFrete(double peso) {
        return estrategia.calcular(peso);
    }
}

// Uso
Pedido pedido = new Pedido();
pedido.setFrete(new Sedex());
pedido.calcularFrete(3.0); // 15.0

pedido.setFrete(new PAC());
pedido.calcularFrete(3.0); // 6.0
```

#### Observer — notificar múltiplos objetos sobre eventos

```java
public interface Observador {
    void atualizar(String evento);
}

public class EventoEstoque {
    private List<Observador> observadores = new ArrayList<>();

    public void assinar(Observador o)  { observadores.add(o); }
    public void cancelar(Observador o) { observadores.remove(o); }

    public void notificar(String evento) {
        observadores.forEach(o -> o.atualizar(evento));
    }

    public void atualizarEstoque(String produto) {
        System.out.println("Estoque atualizado: " + produto);
        notificar("REABASTECIDO: " + produto);
    }
}

// Uso
EventoEstoque estoque = new EventoEstoque();
estoque.assinar(e -> System.out.println("Email: " + e));
estoque.assinar(e -> System.out.println("Push: " + e));

estoque.atualizarEstoque("Notebook");
// Email: REABASTECIDO: Notebook
// Push:  REABASTECIDO: Notebook
```

#### Command — encapsular ações como objetos (suporta undo)

```java
public interface Comando {
    void executar();
    void desfazer();
}

public class LuzAcender implements Comando {
    private Luz luz;
    public LuzAcender(Luz luz) { this.luz = luz; }

    public void executar() { luz.acender(); }
    public void desfazer() { luz.apagar(); }
}

public class ControleRemoto {
    private Deque<Comando> historico = new ArrayDeque<>();

    public void executar(Comando cmd) {
        cmd.executar();
        historico.push(cmd);
    }

    public void desfazer() {
        if (!historico.isEmpty()) historico.pop().desfazer();
    }
}
```

---

### Resumo — qual padrão usar

| Problema | Padrão |
|---|---|
| Preciso de só uma instância | **Singleton** |
| Criação complexa de objetos | **Builder** |
| Decidir qual classe instanciar | **Factory** |
| Adaptar interface incompatível | **Adapter** |
| Adicionar funcionalidade sem herança | **Decorator** |
| Trocar algoritmo em tempo de execução | **Strategy** |
| Notificar múltiplos objetos | **Observer** |
| Histórico de ações / undo | **Command** |