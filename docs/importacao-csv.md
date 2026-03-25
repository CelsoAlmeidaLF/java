# Importação e Exportação de CSV em Java

## Índice

1. [Leitura Manual de CSV](#1-leitura-manual-de-csv)
2. [Leitura com OpenCSV](#2-leitura-com-opencsv)
3. [Exportação para CSV](#3-exportação-para-csv)
4. [Diferentes Separadores](#4-diferentes-separadores)
5. [CSV com Campos entre Aspas](#5-csv-com-campos-entre-aspas)
6. [Tratamento de Erros por Linha](#6-tratamento-de-erros-por-linha)
7. [Boas Práticas](#7-boas-práticas)

---

## 1. Leitura Manual de CSV

### Leitura simples

```java
public List<Map<String, String>> lerCsv(String caminho) throws IOException {
    List<Map<String, String>> registros = new ArrayList<>();
    List<String> linhas = Files.readAllLines(Path.of(caminho));

    if (linhas.isEmpty()) return registros;

    String[] cabecalho = linhas.get(0).split(";");

    for (int i = 1; i < linhas.size(); i++) {
        String linha = linhas.get(i).trim();
        if (linha.isBlank()) continue;

        String[] valores = linha.split(";");
        Map<String, String> registro = new LinkedHashMap<>();

        for (int j = 0; j < cabecalho.length; j++) {
            String valor = j < valores.length ? valores[j].trim() : "";
            registro.put(cabecalho[j].trim(), valor);
        }
        registros.add(registro);
    }
    return registros;
}
```

### Exemplo de arquivo CSV

```
nome;email;valor;status
João Silva;joao@email.com;150.00;ATIVO
Ana Costa;ana@email.com;200.00;ATIVO
Carlos Lima;carlos@email.com;75.50;INATIVO
```

### Leitura linha a linha (eficiente para arquivos grandes)

```java
public void lerCsvGrande(String caminho) throws IOException {
    try (BufferedReader reader = Files.newBufferedReader(Path.of(caminho))) {
        String cabecalhoLinha = reader.readLine();
        String[] cabecalho = cabecalhoLinha.split(";");

        String linha;
        while ((linha = reader.readLine()) != null) {
            if (linha.isBlank()) continue;
            String[] valores = linha.split(";");
            processar(cabecalho, valores);
        }
    }
}
```

---

## 2. Leitura com OpenCSV

### Dependência

```xml
<!-- pom.xml -->
<dependency>
    <groupId>com.opencsv</groupId>
    <artifactId>opencsv</artifactId>
    <version>5.9</version>
</dependency>
```

### Mapeamento direto para objeto

```java
import com.opencsv.bean.*;

// Classe mapeada para o CSV
public class ClienteCSV {

    @CsvBindByName(column = "nome")
    private String nome;

    @CsvBindByName(column = "email")
    private String email;

    @CsvBindByName(column = "valor")
    private double valor;

    @CsvBindByName(column = "status")
    private String status;

    // getters e setters
}

// Leitura com mapeamento automático
public List<ClienteCSV> lerClientes(String caminho) throws IOException {
    try (Reader reader = Files.newBufferedReader(Path.of(caminho))) {
        return new CsvToBeanBuilder<ClienteCSV>(reader)
            .withType(ClienteCSV.class)
            .withSeparator(';')
            .withIgnoreLeadingWhiteSpace(true)
            .withSkipLines(0) // 0 = não pula linhas além do cabeçalho
            .build()
            .parse();
    }
}
```

### Mapeamento por posição (sem cabeçalho)

```java
public class ProdutoCSV {

    @CsvBindByPosition(position = 0)
    private String codigo;

    @CsvBindByPosition(position = 1)
    private String nome;

    @CsvBindByPosition(position = 2)
    private double preco;
}

public List<ProdutoCSV> lerSemCabecalho(String caminho) throws IOException {
    try (Reader reader = Files.newBufferedReader(Path.of(caminho))) {
        return new CsvToBeanBuilder<ProdutoCSV>(reader)
            .withType(ProdutoCSV.class)
            .withSeparator(';')
            .build()
            .parse();
    }
}
```

---

## 3. Exportação para CSV

### Exportação simples

```java
public void exportarCsv(List<Produto> produtos, String caminho) throws IOException {
    StringBuilder sb = new StringBuilder();

    // Cabeçalho
    sb.append("nome;preco;categoria\n");

    // Dados
    for (Produto p : produtos) {
        sb.append(p.getNome()).append(";")
          .append(p.getPreco()).append(";")
          .append(p.getCategoria()).append("\n");
    }

    Files.writeString(Path.of(caminho), sb.toString(), StandardCharsets.UTF_8);
}
```

### Exportação com OpenCSV

```java
import com.opencsv.CSVWriter;
import com.opencsv.bean.StatefulBeanToCsv;
import com.opencsv.bean.StatefulBeanToCsvBuilder;

// Exportar lista de objetos diretamente
public void exportarComOpenCsv(List<ClienteCSV> clientes, String caminho) throws Exception {
    try (Writer writer = Files.newBufferedWriter(Path.of(caminho), StandardCharsets.UTF_8)) {
        StatefulBeanToCsv<ClienteCSV> beanToCsv = new StatefulBeanToCsvBuilder<ClienteCSV>(writer)
            .withSeparator(';')
            .build();
        beanToCsv.write(clientes);
    }
}
```

### Exportação com controle total (CSVWriter)

```java
public void exportarManual(List<Produto> produtos, String caminho) throws IOException {
    try (CSVWriter writer = new CSVWriter(
            Files.newBufferedWriter(Path.of(caminho), StandardCharsets.UTF_8),
            ';',                     // separador
            CSVWriter.NO_QUOTE_CHARACTER,
            CSVWriter.DEFAULT_ESCAPE_CHARACTER,
            CSVWriter.DEFAULT_LINE_END)) {

        // Cabeçalho
        writer.writeNext(new String[]{"nome", "preco", "categoria"});

        // Dados
        for (Produto p : produtos) {
            writer.writeNext(new String[]{
                p.getNome(),
                String.valueOf(p.getPreco()),
                p.getCategoria()
            });
        }
    }
}
```

---

## 4. Diferentes Separadores

```java
// Separado por vírgula (padrão internacional)
String[] valores = linha.split(",");

// Separado por ponto e vírgula (padrão Brasil)
String[] valores = linha.split(";");

// Separado por pipe
String[] valores = linha.split("\\|");

// Separado por tab (TSV)
String[] valores = linha.split("\t");

// Detectar separador automaticamente
public char detectarSeparador(String primeiraLinha) {
    int virgulas     = primeiraLinha.split(",").length;
    int pontoVirgula = primeiraLinha.split(";").length;
    int pipes        = primeiraLinha.split("\\|").length;

    if (pontoVirgula >= virgulas && pontoVirgula >= pipes) return ';';
    if (pipes >= virgulas) return '|';
    return ',';
}
```

---

## 5. CSV com Campos entre Aspas

Campos que contêm o separador devem estar entre aspas:

```
nome;endereco;valor
"João Silva";"Rua A, 123";150.00
"Ana; Costa";"Rua B, 456";200.00
```

```java
// OpenCSV trata aspas automaticamente
// Leitura manual — parsear respeitando aspas
public String[] parsearLinha(String linha, char separador) {
    List<String> campos = new ArrayList<>();
    boolean dentroAspas = false;
    StringBuilder campo = new StringBuilder();

    for (char c : linha.toCharArray()) {
        if (c == '"') {
            dentroAspas = !dentroAspas;
        } else if (c == separador && !dentroAspas) {
            campos.add(campo.toString().trim());
            campo = new StringBuilder();
        } else {
            campo.append(c);
        }
    }
    campos.add(campo.toString().trim()); // último campo

    return campos.toArray(new String[0]);
}
```

---

## 6. Tratamento de Erros por Linha

```java
public class ResultadoImportacao<T> {
    private List<T> sucesso = new ArrayList<>();
    private List<String> erros = new ArrayList<>();

    public void adicionarSucesso(T item) { sucesso.add(item); }
    public void adicionarErro(String msg) { erros.add(msg); }

    public List<T> getSucesso()    { return sucesso; }
    public List<String> getErros() { return erros; }
    public boolean temErros()      { return !erros.isEmpty(); }
}

public ResultadoImportacao<Cliente> importarClientes(String caminho) throws IOException {
    ResultadoImportacao<Cliente> resultado = new ResultadoImportacao<>();

    try (BufferedReader reader = Files.newBufferedReader(Path.of(caminho))) {
        reader.readLine(); // pula cabeçalho
        int numeroLinha = 2;
        String linha;

        while ((linha = reader.readLine()) != null) {
            if (linha.isBlank()) { numeroLinha++; continue; }

            try {
                String[] campos = linha.split(";");

                String nome  = campos[0].trim();
                String email = campos[1].trim();
                double valor = Double.parseDouble(campos[2].trim());

                if (nome.isBlank())       throw new IllegalArgumentException("Nome vazio");
                if (!email.contains("@")) throw new IllegalArgumentException("Email inválido");
                if (valor < 0)            throw new IllegalArgumentException("Valor negativo");

                resultado.adicionarSucesso(new Cliente(nome, email, valor));

            } catch (Exception e) {
                resultado.adicionarErro("Linha " + numeroLinha + ": " + e.getMessage());
            }
            numeroLinha++;
        }
    }
    return resultado;
}
```

---

## 7. Boas Práticas

```
✔ Sempre especificar o charset (UTF-8) na leitura e escrita
✔ Verificar se o arquivo tem cabeçalho antes de processar
✔ Tratar erros linha a linha — não abortar tudo por uma linha inválida
✔ Usar OpenCSV para arquivos com aspas ou separadores dentro dos campos
✔ Detectar o separador automaticamente quando não for fixo
✔ Fazer backup antes de processar
✔ Registrar log de erros com número da linha
```

### Fluxo completo recomendado

```java
public void importar(String caminho) throws IOException {
    Path arquivo = Path.of(caminho);

    // 1. Validar extensão
    if (!caminho.endsWith(".csv")) throw new IllegalArgumentException("Arquivo inválido");

    // 2. Backup
    Files.copy(arquivo, Path.of("backup/" + arquivo.getFileName()),
               StandardCopyOption.REPLACE_EXISTING);

    // 3. Importar com validação
    ResultadoImportacao<Cliente> resultado = importarClientes(caminho);

    // 4. Salvar registros válidos
    salvarNoBanco(resultado.getSucesso());

    // 5. Log do resultado
    System.out.printf("Importados: %d | Erros: %d%n",
        resultado.getSucesso().size(), resultado.getErros().size());
    resultado.getErros().forEach(System.out::println);

    // 6. Mover arquivo processado
    Files.move(arquivo, Path.of("processados/" + arquivo.getFileName()),
               StandardCopyOption.REPLACE_EXISTING);
}
```
