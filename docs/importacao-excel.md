# Importação e Exportação de Excel em Java

## Índice

1. [Dependência Apache POI](#1-dependência-apache-poi)
2. [Leitura de Excel](#2-leitura-de-excel)
3. [Exportação para Excel](#3-exportação-para-excel)
4. [Formatação e Estilos](#4-formatação-e-estilos)
5. [Múltiplas Abas](#5-múltiplas-abas)
6. [Tratamento de Erros por Linha](#6-tratamento-de-erros-por-linha)
7. [Boas Práticas](#7-boas-práticas)

---

## 1. Dependência Apache POI

```xml
<!-- pom.xml -->
<dependency>
    <groupId>org.apache.poi</groupId>
    <artifactId>poi-ooxml</artifactId>
    <version>5.2.5</version>
</dependency>
```

---

## 2. Leitura de Excel

### Leitura simples de .xlsx

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
            if (isLinhaVazia(row)) continue;

            String nome      = row.getCell(0).getStringCellValue();
            double preco     = row.getCell(1).getNumericCellValue();
            String categoria = row.getCell(2).getStringCellValue();

            produtos.add(new Produto(nome, preco, categoria));
        }
    }
    return produtos;
}

private boolean isLinhaVazia(Row row) {
    for (Cell cell : row) {
        if (cell != null && cell.getCellType() != CellType.BLANK) return false;
    }
    return true;
}
```

### Leitura segura — qualquer tipo de célula

```java
private String lerCelula(Cell cell) {
    if (cell == null) return "";

    return switch (cell.getCellType()) {
        case STRING  -> cell.getStringCellValue().trim();
        case NUMERIC -> DateUtil.isCellDateFormatted(cell)
                        ? cell.getLocalDateTimeCellValue().toLocalDate().toString()
                        : String.valueOf((long) cell.getNumericCellValue());
        case BOOLEAN -> String.valueOf(cell.getBooleanCellValue());
        case FORMULA -> lerCelulaFormula(cell);
        case BLANK   -> "";
        default      -> "";
    };
}

private String lerCelulaFormula(Cell cell) {
    FormulaEvaluator evaluator = cell.getSheet()
        .getWorkbook().getCreationHelper().createFormulaEvaluator();
    CellValue valor = evaluator.evaluate(cell);

    return switch (valor.getCellType()) {
        case STRING  -> valor.getStringValue();
        case NUMERIC -> String.valueOf((long) valor.getNumberValue());
        case BOOLEAN -> String.valueOf(valor.getBooleanValue());
        default      -> "";
    };
}
```

### Mapeamento direto para objeto

```java
public class ProdutoExcelMapper {

    public Produto mapear(Row row) {
        return new Produto(
            lerCelula(row.getCell(0)),                    // nome
            Double.parseDouble(lerCelula(row.getCell(1))), // preco
            lerCelula(row.getCell(2))                     // categoria
        );
    }

    public List<Produto> lerTodos(Sheet sheet) {
        List<Produto> lista = new ArrayList<>();
        Iterator<Row> iterator = sheet.iterator();
        if (iterator.hasNext()) iterator.next(); // pula cabeçalho

        while (iterator.hasNext()) {
            Row row = iterator.next();
            if (!isLinhaVazia(row)) {
                lista.add(mapear(row));
            }
        }
        return lista;
    }
}
```

---

## 3. Exportação para Excel

### Exportação básica

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

    // Auto-ajustar largura das colunas
    for (int i = 0; i < 3; i++) sheet.autoSizeColumn(i);

    try (FileOutputStream out = new FileOutputStream(caminho)) {
        workbook.write(out);
    }
    workbook.close();
}
```

---

## 4. Formatação e Estilos

### Cabeçalho estilizado + células numéricas formatadas

```java
public void exportarComEstilo(List<Produto> produtos, String caminho) throws IOException {
    Workbook workbook = new XSSFWorkbook();
    Sheet sheet = workbook.createSheet("Produtos");

    // Estilo do cabeçalho
    CellStyle estiloHeader = workbook.createCellStyle();
    Font fonteBold = workbook.createFont();
    fonteBold.setBold(true);
    fonteBold.setFontHeightInPoints((short) 12);
    estiloHeader.setFont(fonteBold);
    estiloHeader.setFillForegroundColor(IndexedColors.LIGHT_BLUE.getIndex());
    estiloHeader.setFillPattern(FillPatternType.SOLID_FOREGROUND);
    estiloHeader.setAlignment(HorizontalAlignment.CENTER);

    // Estilo monetário
    CellStyle estiloMoeda = workbook.createCellStyle();
    DataFormat formato = workbook.createDataFormat();
    estiloMoeda.setDataFormat(formato.getFormat("R$ #,##0.00"));

    // Cabeçalho
    Row header = sheet.createRow(0);
    String[] colunas = {"Nome", "Preço", "Categoria"};
    for (int i = 0; i < colunas.length; i++) {
        Cell cell = header.createCell(i);
        cell.setCellValue(colunas[i]);
        cell.setCellStyle(estiloHeader);
    }

    // Dados
    int rowIndex = 1;
    for (Produto p : produtos) {
        Row row = sheet.createRow(rowIndex++);
        row.createCell(0).setCellValue(p.getNome());

        Cell cellPreco = row.createCell(1);
        cellPreco.setCellValue(p.getPreco());
        cellPreco.setCellStyle(estiloMoeda);

        row.createCell(2).setCellValue(p.getCategoria());
    }

    for (int i = 0; i < 3; i++) sheet.autoSizeColumn(i);

    try (FileOutputStream out = new FileOutputStream(caminho)) {
        workbook.write(out);
    }
    workbook.close();
}
```

---

## 5. Múltiplas Abas

```java
public void exportarMultiplasAbas(Map<String, List<Produto>> dadosPorCategoria,
                                   String caminho) throws IOException {
    Workbook workbook = new XSSFWorkbook();

    for (Map.Entry<String, List<Produto>> entrada : dadosPorCategoria.entrySet()) {
        Sheet sheet = workbook.createSheet(entrada.getKey());
        List<Produto> produtos = entrada.getValue();

        Row header = sheet.createRow(0);
        header.createCell(0).setCellValue("Nome");
        header.createCell(1).setCellValue("Preço");

        int rowIndex = 1;
        for (Produto p : produtos) {
            Row row = sheet.createRow(rowIndex++);
            row.createCell(0).setCellValue(p.getNome());
            row.createCell(1).setCellValue(p.getPreco());
        }

        sheet.autoSizeColumn(0);
        sheet.autoSizeColumn(1);
    }

    try (FileOutputStream out = new FileOutputStream(caminho)) {
        workbook.write(out);
    }
    workbook.close();
}
```

---

## 6. Tratamento de Erros por Linha

```java
public class ResultadoImportacao {
    private List<Produto> sucesso = new ArrayList<>();
    private List<String> erros = new ArrayList<>();

    public void adicionarSucesso(Produto p) { sucesso.add(p); }
    public void adicionarErro(String msg)    { erros.add(msg); }

    public int total()    { return sucesso.size() + erros.size(); }
    public boolean temErros() { return !erros.isEmpty(); }
}

public ResultadoImportacao importarComValidacao(String caminho) throws IOException {
    ResultadoImportacao resultado = new ResultadoImportacao();

    try (Workbook workbook = new XSSFWorkbook(new FileInputStream(caminho))) {
        Sheet sheet = workbook.getSheetAt(0);
        int numeroLinha = 1;

        for (Row row : sheet) {
            if (numeroLinha == 1) { numeroLinha++; continue; }
            if (isLinhaVazia(row)) continue;

            try {
                String nome  = lerCelula(row.getCell(0));
                double preco = Double.parseDouble(lerCelula(row.getCell(1)));

                if (nome.isBlank()) throw new IllegalArgumentException("Nome vazio");
                if (preco <= 0)     throw new IllegalArgumentException("Preço inválido");

                resultado.adicionarSucesso(new Produto(nome, preco, lerCelula(row.getCell(2))));
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
✔ Sempre fechar o Workbook com try-with-resources
✔ Verificar o tipo da célula antes de ler (evita exceções)
✔ Pular linhas vazias
✔ Validar dados linha a linha (não abortar tudo por uma linha inválida)
✔ Auto-ajustar colunas após preencher os dados
✔ Limitar tamanho máximo do arquivo aceito
✔ Fazer backup do arquivo antes de processar
```

### Fluxo completo recomendado

```java
public void importar(String caminho) throws IOException {
    Path arquivo = Path.of(caminho);

    // 1. Validar extensão
    if (!caminho.endsWith(".xlsx")) throw new IllegalArgumentException("Arquivo inválido");

    // 2. Backup
    Files.copy(arquivo, Path.of("backup/" + arquivo.getFileName()));

    // 3. Importar com validação
    ResultadoImportacao resultado = importarComValidacao(caminho);

    // 4. Salvar no banco
    salvarNoBanco(resultado.getSucesso());

    // 5. Relatório
    System.out.println("Importados: " + resultado.getSucesso().size());
    System.out.println("Erros: "      + resultado.getErros().size());
    resultado.getErros().forEach(System.out::println);

    // 6. Mover arquivo
    Files.move(arquivo, Path.of("processados/" + arquivo.getFileName()));
}
```
