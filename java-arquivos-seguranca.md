# Java — Arquivos, Importação e Segurança de Dados

## Índice

1. [I/O e Arquivos com NIO](#1-io-e-arquivos-com-nio)
2. [Importação de CSV](#2-importação-de-csv)
3. [Importação de TXT](#3-importação-de-txt)
4. [Importação de Excel](#4-importação-de-excel)
5. [Processamento Assíncrono de Arquivos](#5-processamento-assíncrono-de-arquivos)
6. [Monitoramento de Pasta](#6-monitoramento-de-pasta)
7. [Segurança e Criptografia](#7-segurança-e-criptografia)
8. [Boas Práticas](#8-boas-práticas)

---

## 1. I/O e Arquivos com NIO

### Operações básicas

```java
import java.nio.file.*;

Path arquivo = Path.of("dados/arquivo.txt");

// Ler tudo de uma vez
String conteudo = Files.readString(arquivo);
List<String> linhas = Files.readAllLines(arquivo);

// Escrever
Files.writeString(arquivo, "conteúdo");
Files.write(arquivo, List.of("linha1", "linha2"));

// Verificar existência e criar diretórios
boolean existe = Files.exists(arquivo);
Files.createDirectories(Path.of("dados/importados"));

// Copiar e mover
Files.copy(arquivo, Path.of("backup/arquivo.txt"));
Files.move(arquivo, Path.of("processados/arquivo.txt"));

// Deletar
Files.delete(arquivo);
Files.deleteIfExists(arquivo);
```

### Ler arquivos grandes (linha a linha — eficiente em memória)

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
// Todos os arquivos de uma pasta
Files.list(Path.of("importacoes"))
     .filter(p -> p.toString().endsWith(".csv"))
     .forEach(System.out::println);

// Recursivo — subpastas também
Files.walk(Path.of("importacoes"))
     .filter(Files::isRegularFile)
     .filter(p -> p.toString().endsWith(".xlsx"))
     .forEach(System.out::println);
```

---

## 2. Importação de CSV

### Leitura manual (sem biblioteca)

```java
public class CsvReader {

    public List<Map<String, String>> ler(String caminho) throws IOException {
        List<Map<String, String>> registros = new ArrayList<>();
        List<String> linhas = Files.readAllLines(Path.of(caminho));

        String[] cabecalho = linhas.get(0).split(";");

        for (int i = 1; i < linhas.size(); i++) {
            String[] valores = linhas.get(i).split(";");
            Map<String, String> registro = new LinkedHashMap<>();

            for (int j = 0; j < cabecalho.length; j++) {
                registro.put(cabecalho[j].trim(), valores[j].trim());
            }
            registros.add(registro);
        }
        return registros;
    }
}

// Uso
// arquivo.csv:
// nome;email;valor
// João;joao@email.com;150.00
// Ana;ana@email.com;200.00

CsvReader reader = new CsvReader();
List<Map<String, String>> dados = reader.ler("arquivo.csv");

for (Map<String, String> linha : dados) {
    System.out.println(linha.get("nome") + " → " + linha.get("valor"));
}
```

### Leitura com OpenCSV (recomendado — lida com vírgulas e aspas nos dados)

```xml
<!-- pom.xml -->
<dependency>
    <groupId>com.opencsv</groupId>
    <artifactId>opencsv</artifactId>
    <version>5.9</version>
</dependency>
```

```java
import com.opencsv.CSVReader;
import com.opencsv.bean.CsvToBean;
import com.opencsv.bean.CsvToBeanBuilder;

// Mapear diretamente para um objeto
public class ProdutoCSV {
    @CsvBindByName(column = "nome")
    private String nome;

    @CsvBindByName(column = "preco")
    private double preco;

    @CsvBindByName(column = "categoria")
    private String categoria;
}

// Leitura e mapeamento automático
try (Reader reader = Files.newBufferedReader(Path.of("produtos.csv"))) {
    List<ProdutoCSV> produtos = new CsvToBeanBuilder<ProdutoCSV>(reader)
        .withType(ProdutoCSV.class)
        .withSeparator(';')
        .withIgnoreLeadingWhiteSpace(true)
        .build()
        .parse();

    produtos.forEach(p -> System.out.println(p.getNome()));
}
```

### Exportar para CSV

```java
public void exportar(List<Produto> produtos, String caminho) throws IOException {
    StringBuilder sb = new StringBuilder();
    sb.append("nome;preco;categoria\n"); // cabeçalho

    for (Produto p : produtos) {
        sb.append(p.getNome()).append(";")
          .append(p.getPreco()).append(";")
          .append(p.getCategoria()).append("\n");
    }

    Files.writeString(Path.of(caminho), sb.toString());
}
```

---

## 3. Importação de TXT

### Largura fixa (posicional)

Muito comum em integrações bancárias e legados.

```java
// Exemplo de layout posicional:
// Posição 0-9:   código (10 chars)
// Posição 10-39: nome   (30 chars)
// Posição 40-49: valor  (10 chars)

public class TxtPositionalReader {

    public List<Registro> ler(String caminho) throws IOException {
        return Files.readAllLines(Path.of(caminho)).stream()
            .filter(linha -> !linha.isBlank())
            .map(this::parseLinha)
            .collect(Collectors.toList());
    }

    private Registro parseLinha(String linha) {
        String codigo = linha.substring(0, 10).trim();
        String nome   = linha.substring(10, 40).trim();
        String valor  = linha.substring(40, 50).trim();

        return new Registro(codigo, nome, Double.parseDouble(valor));
    }
}
```

### Delimitado por pipe ou outro separador

```java
// linha: 001|João Silva|150.00|ATIVO
String[] partes = linha.split("\\|");
String codigo = partes[0];
String nome   = partes[1];
double valor  = Double.parseDouble(partes[2]);
String status = partes[3];
```

### Com identificador de tipo de registro (CNAB)

```java
// Padrão CNAB — primeiro char indica o tipo da linha
// 0 = Header, 1 = Detalhe, 9 = Trailer

public void processar(String caminho) throws IOException {
    Files.readAllLines(Path.of(caminho)).forEach(linha -> {
        char tipo = linha.charAt(0);
        switch (tipo) {
            case '0' -> processarHeader(linha);
            case '1' -> processarDetalhe(linha);
            case '9' -> processarTrailer(linha);
        }
    });
}
```

---

## 4. Importação de Excel

### Dependência Apache POI

```xml
<!-- pom.xml -->
<dependency>
    <groupId>org.apache.poi</groupId>
    <artifactId>poi-ooxml</artifactId>
    <version>5.2.5</version>
</dependency>
```

### Ler arquivo .xlsx

```java
import org.apache.poi.ss.usermodel.*;
import org.apache.poi.xssf.usermodel.XSSFWorkbook;

public List<Produto> lerExcel(String caminho) throws IOException {
    List<Produto> produtos = new ArrayList<>();

    try (Workbook workbook = new XSSFWorkbook(new FileInputStream(caminho))) {
        Sheet sheet = workbook.getSheetAt(0); // primeira aba

        boolean primeiraLinha = true;
        for (Row row : sheet) {
            if (primeiraLinha) { primeiraLinha = false; continue; } // pula cabeçalho

            String nome      = row.getCell(0).getStringCellValue();
            double preco     = row.getCell(1).getNumericCellValue();
            String categoria = row.getCell(2).getStringCellValue();

            produtos.add(new Produto(nome, preco, categoria));
        }
    }
    return produtos;
}
```

### Ler com tipo de célula dinâmico (seguro)

```java
private String lerCelula(Cell cell) {
    if (cell == null) return "";

    return switch (cell.getCellType()) {
        case STRING  -> cell.getStringCellValue().trim();
        case NUMERIC -> DateUtil.isCellDateFormatted(cell)
                        ? cell.getLocalDateTimeCellValue().toString()
                        : String.valueOf((long) cell.getNumericCellValue());
        case BOOLEAN -> String.valueOf(cell.getBooleanCellValue());
        case FORMULA -> cell.getCachedFormulaResultType().name();
        default      -> "";
    };
}
```

### Exportar para Excel

```java
public void exportarExcel(List<Produto> produtos, String caminho) throws IOException {
    Workbook workbook = new XSSFWorkbook();
    Sheet sheet = workbook.createSheet("Produtos");

    // Cabeçalho
    Row header = sheet.createRow(0);
    header.createCell(0).setCellValue("Nome");
    header.createCell(1).setCellValue("Preço");
    header.createCell(2).setCellValue("Categoria");

    // Dados
    int rowIndex = 1;
    for (Produto p : produtos) {
        Row row = sheet.createRow(rowIndex++);
        row.createCell(0).setCellValue(p.getNome());
        row.createCell(1).setCellValue(p.getPreco());
        row.createCell(2).setCellValue(p.getCategoria());
    }

    // Auto-ajustar colunas
    for (int i = 0; i < 3; i++) sheet.autoSizeColumn(i);

    try (FileOutputStream out = new FileOutputStream(caminho)) {
        workbook.write(out);
    }
    workbook.close();
}
```

---

## 5. Processamento Assíncrono de Arquivos

### Processar múltiplos arquivos em paralelo

```java
public void processarPasta(String pasta) throws Exception {
    List<Path> arquivos = Files.list(Path.of(pasta))
        .filter(p -> p.toString().endsWith(".csv"))
        .collect(Collectors.toList());

    // Processa todos em paralelo
    List<CompletableFuture<Void>> tarefas = arquivos.stream()
        .map(arquivo -> CompletableFuture.runAsync(() -> {
            try {
                processarArquivo(arquivo);
                moverParaProcessados(arquivo);
            } catch (Exception e) {
                moverParaErros(arquivo, e);
            }
        }))
        .collect(Collectors.toList());

    // Aguarda todos terminarem
    CompletableFuture.allOf(tarefas.toArray(new CompletableFuture[0])).join();
}

private void moverParaProcessados(Path arquivo) throws IOException {
    Path destino = arquivo.getParent().resolve("processados").resolve(arquivo.getFileName());
    Files.createDirectories(destino.getParent());
    Files.move(arquivo, destino);
}

private void moverParaErros(Path arquivo, Exception e) {
    try {
        Path destino = arquivo.getParent().resolve("erros").resolve(arquivo.getFileName());
        Files.createDirectories(destino.getParent());
        Files.move(arquivo, destino);
        log.error("Erro ao processar {}: {}", arquivo.getFileName(), e.getMessage());
    } catch (IOException ex) {
        log.error("Falha ao mover arquivo para erros: {}", ex.getMessage());
    }
}
```

---

## 6. Monitoramento de Pasta

Detecta automaticamente quando um arquivo novo é adicionado.

```java
public class MonitorPasta implements Runnable {

    private final Path pasta;

    public MonitorPasta(String caminho) {
        this.pasta = Path.of(caminho);
    }

    @Override
    public void run() {
        try (WatchService watcher = FileSystems.getDefault().newWatchService()) {
            pasta.register(watcher,
                StandardWatchEventKinds.ENTRY_CREATE,
                StandardWatchEventKinds.ENTRY_MODIFY);

            System.out.println("Monitorando: " + pasta);

            while (true) {
                WatchKey key = watcher.take(); // aguarda evento

                for (WatchEvent<?> evento : key.pollEvents()) {
                    Path arquivo = pasta.resolve((Path) evento.context());
                    System.out.println("Novo arquivo detectado: " + arquivo);
                    processarArquivo(arquivo);
                }
                key.reset();
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}

// Uso — rodar em thread separada
new Thread(new MonitorPasta("importacoes/entrada")).start();
```

---

## 7. Segurança e Criptografia

### 7.1 Hash — verificar integridade de arquivos

```java
import java.security.MessageDigest;

// Gerar hash SHA-256 de um arquivo
public String hashArquivo(String caminho) throws Exception {
    MessageDigest digest = MessageDigest.getInstance("SHA-256");
    byte[] bytes = Files.readAllBytes(Path.of(caminho));
    byte[] hash = digest.digest(bytes);

    // Converter para hex
    StringBuilder hex = new StringBuilder();
    for (byte b : hash) {
        hex.append(String.format("%02x", b));
    }
    return hex.toString();
}

// Verificar se arquivo foi alterado
public boolean arquivoIntacto(String caminho, String hashOriginal) throws Exception {
    return hashOriginal.equals(hashArquivo(caminho));
}
```

### 7.2 Criptografia AES — cifrar dados sensíveis

```java
import javax.crypto.*;
import javax.crypto.spec.*;

public class CriptografiaAES {

    private static final String ALGORITMO = "AES/CBC/PKCS5Padding";

    // Gerar chave (guardar em local seguro / variável de ambiente)
    public static SecretKey gerarChave() throws Exception {
        KeyGenerator kg = KeyGenerator.getInstance("AES");
        kg.init(256);
        return kg.generateKey();
    }

    // Cifrar
    public static byte[] cifrar(String texto, SecretKey chave) throws Exception {
        Cipher cipher = Cipher.getInstance(ALGORITMO);
        byte[] iv = new byte[16];
        new java.security.SecureRandom().nextBytes(iv);
        cipher.init(Cipher.ENCRYPT_MODE, chave, new IvParameterSpec(iv));

        byte[] textoCifrado = cipher.doFinal(texto.getBytes());

        // Retorna IV + dados cifrados juntos
        byte[] resultado = new byte[iv.length + textoCifrado.length];
        System.arraycopy(iv, 0, resultado, 0, iv.length);
        System.arraycopy(textoCifrado, 0, resultado, iv.length, textoCifrado.length);
        return resultado;
    }

    // Decifrar
    public static String decifrar(byte[] dados, SecretKey chave) throws Exception {
        Cipher cipher = Cipher.getInstance(ALGORITMO);

        byte[] iv = Arrays.copyOfRange(dados, 0, 16);
        byte[] textoCifrado = Arrays.copyOfRange(dados, 16, dados.length);

        cipher.init(Cipher.DECRYPT_MODE, chave, new IvParameterSpec(iv));
        return new String(cipher.doFinal(textoCifrado));
    }
}

// Uso
SecretKey chave = CriptografiaAES.gerarChave();

byte[] cifrado = CriptografiaAES.cifrar("dado sensível", chave);
String original = CriptografiaAES.decifrar(cifrado, chave);
```

### 7.3 Cifrar arquivo inteiro

```java
public void cifrarArquivo(String entrada, String saida, SecretKey chave) throws Exception {
    byte[] conteudo = Files.readAllBytes(Path.of(entrada));
    byte[] cifrado = CriptografiaAES.cifrar(new String(conteudo), chave);
    Files.write(Path.of(saida), cifrado);
}

public void decifrarArquivo(String entrada, String saida, SecretKey chave) throws Exception {
    byte[] cifrado = Files.readAllBytes(Path.of(entrada));
    String conteudo = CriptografiaAES.decifrar(cifrado, chave);
    Files.writeString(Path.of(saida), conteudo);
}
```

### 7.4 Variáveis de ambiente — nunca expor chaves no código

```java
// ERRADO — nunca faça isso
String chave = "minha-chave-secreta-123";

// CORRETO — ler do ambiente
String chave = System.getenv("APP_SECRET_KEY");
String dbUrl  = System.getenv("DATABASE_URL");

// Com valor padrão para desenvolvimento
String porta = Optional.ofNullable(System.getenv("APP_PORT")).orElse("8080");
```

```bash
# .env (nunca suba para o git!)
APP_SECRET_KEY=sua-chave-aqui
DATABASE_URL=jdbc:postgresql://localhost:5432/meubanco
```

---

## 8. Boas Práticas

### Estrutura de pastas para workflow de arquivos

```
importacoes/
├── entrada/        # arquivos aguardando processamento
├── processados/    # arquivos processados com sucesso
├── erros/          # arquivos com falha
└── backup/         # cópias de segurança
```

### Fluxo seguro de importação

```java
public void importar(Path arquivo) {
    Path backup = fazerBackup(arquivo);           // 1. fazer backup primeiro
    validarArquivo(arquivo);                       // 2. validar formato e integridade
    List<?> registros = parsearArquivo(arquivo);  // 3. converter para objetos
    validarRegistros(registros);                   // 4. validar regras de negócio
    salvarNoBanco(registros);                      // 5. persistir
    moverParaProcessados(arquivo);                 // 6. mover arquivo
    registrarLog(arquivo, registros.size());       // 7. registrar auditoria
}
```

### Checklist de segurança

- [ ] Nunca salvar chaves/senhas no código — usar variáveis de ambiente
- [ ] Sempre fazer backup antes de processar
- [ ] Validar hash do arquivo antes de importar (integridade)
- [ ] Mover arquivos processados para fora da pasta de entrada
- [ ] Registrar log de auditoria (quem importou, quando, quantos registros)
- [ ] Tratar e registrar erros linha a linha (não abortar tudo por uma linha inválida)
- [ ] Limitar tamanho máximo de arquivo aceito
- [ ] Sanitizar dados de entrada (evitar injeção de dados)
