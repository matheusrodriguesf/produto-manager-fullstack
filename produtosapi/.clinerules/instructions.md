# Regras e Padrões do Projeto — Produtos API

## 1. Objetivo

Você é um desenvolvedor Java especializado no ecossistema Spring Boot.

Ao modificar, refatorar, corrigir ou criar código neste projeto, **siga rigorosamente as regras deste documento**.

Antes de realizar alterações:

1. Analise a estrutura existente do projeto.
2. Reutilize padrões já existentes quando forem compatíveis com estas regras.
3. Não introduza tecnologias ou padrões arquiteturais desnecessários.
4. Preserve a compatibilidade com o código existente.
5. Evite alterações fora do escopo solicitado.

---

## 2. Stack Tecnológica

O projeto utiliza:

* **Java 21**
* **Spring Boot 4.1.1**
* **Maven**
* **Spring Web**
* **Spring Data JPA**
* **H2 Database**
* **Lombok**

### 2.1 Java

Utilize Java 21 e seus recursos modernos quando trouxerem benefício real ao código.

Priorize, quando apropriado:

* `var`
* Pattern Matching
* `switch` moderno
* `Record`
* Text Blocks
* Streams
* Optional

Matenha sempre a legibilidade e simplicidade.

### 2.2 Maven

O projeto utiliza Maven.

Regras:

* Utilize exclusivamente `pom.xml`.
* **Nunca adicione arquivos do Gradle**, como `build.gradle` ou `settings.gradle`.
* Ao adicionar uma dependência, altere o `pom.xml` existente.
* Mantenha a organização atual das dependências.
* Não adicione dependências quando a funcionalidade puder ser implementada utilizando recursos já disponíveis no projeto.

### 2.3 Banco de Dados

O banco utilizado no ambiente de desenvolvimento/testes é o **H2 Database**.

Utilize JPA/Spring Data JPA para acesso aos dados.

---

# 3. Arquitetura

O projeto utiliza uma arquitetura baseada em camadas.

```
br.com.arcelino.produtosapi
├── controller
├── entity
├── dto
├── repository
├── service
├── config
└── ProdutosapiApplication.java
```

## 3.1 Arquitetura do Projeto e Camadas

Mantenha estritamente a separação de responsabilidades utilizando o seguinte fluxo estrutural:

### 3.1.1 Entity

- Entidades representam os objetos de domínio do sistema. 
- Entidades devem ser nomeadas com substantivos no singular (ex: `Produto`, `Categoria`).
- As entidades devem ser colocadas no pacote `entity`.
- As tabelas no banco de dados devem ser nomeadas com o prefixo `tb_` seguido do nome da entidade (ex: `tb_produto`, `tb_categoria`). O nome da tabela pode ser especificado usando a anotação `@Table(name = "nome_da_tabela")` na classe da entidade.

Padrões de Entidade:

```java
@Entity
@Table(name = "table_name")
@Builder
@Getter
@Setter
@NoArgsConstructor
@AllArgsConstructor
@FieldDefaults(level = AccessLevel.PRIVATE)
public class EntityName {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    Long id;

    // Outros atributos da entidade

}
```

### 3.1.2 DTO

- As classes DTO (Data Transfer Object) devem ser nomeadas com o sufixo `DTO` (ex: `ProdutoDTO`, `CategoriaDTO`). Elas são usadas para transferir dados entre a camada de apresentação e a camada de serviço, evitando expor diretamente as entidades do banco de dados.
- Solicitação: `{Entity}RequestDTO` (ex.: `ProdutoRequestDTO`)
- Resposta: `{Entity}ResponseDTO` (ex.: `ProdutoResponseDTO`)

Exemplo de Request:

```java
public record ProdutoRequestDTO(
    String nome,
    BigDecimal preco
) {
}
```
Exemplo de Response:
```java
public record ProdutoResponseDTO(
    Long id,
    String nome,
    BigDecimal preco
) {
}
```

### 3.1.3 Controller

- Nome do Controller: O nome do controller deve ser o nome da entidade seguido de `Controller` (ex: `ProdutoController`, `CategoriaController`).
- Pacote: Os controllers devem ser colocados no pacote `controller`.
- Anotação: Cada controller deve ser anotado com `@RestController` e `@RequestMapping("/nome_da_entidade")`, no plural onde `nome_da_entidade` é o nome da entidade em minúsculas (ex: `/produtos`, `/categorias`).

Estrutura da controller
```java
@RestController
@RequestMapping("/produtos")
@FieldDefaults(level = AccessLevel.PRIVATE, makeFinal = true)
@RequiredArgsConstructor
public class ProdutoController {

    ProdutoService produtoService;

    // Endpoints
}
```
Padrão de Endpoints

| Operação      | Método | Endpoint |
| ------------- | ------ | -------- |
| Criar         | POST   | `/`      |
| Buscar por ID | GET    | `/{id}`  |
| Buscar todos  | GET    | `/`      |
| Atualizar     | PUT    | `/{id}`  |
| Excluir       | DELETE | `/{id}`  |

### 3.1.4 Service

- Nome do Service: O nome do service deve ser o nome da entidade seguido de `Service` (ex: `ProdutoService`, `CategoriaService`).
- Pacote: Os services devem ser colocados no pacote `service`.
- Anotação: Cada service deve ser anotado com `@Service`. A camada de serviço é responsável por implementar a lógica de negócio da aplicação.

Padrões da Camada de Serviço 

```java 
@Slf4j
@Service
@FieldDefaults(level = AccessLevel.PRIVATE, makeFinal = true)
@RequiredArgsConstructor 
public class EntityService { 
    

    EntityRepository entityRepository; 
    
    RelatedRepository relatedRepository; 
    
    // Métodos públicos com lógica de negócios clara 
} 
``` 


#### 3.1.5 Repository
- A camada de repositório é responsável por interagir com o banco de dados, realizando operações CRUD (Create, Read, Update, Delete) nas entidades.
- Nome do Repository: O nome do repository deve ser o nome da entidade seguido de `Repository` (ex: `ProdutoRepository`, `CategoriaRepository`).
- Pacote: Os repositories devem ser colocados no pacote `repository`.
- Anotação: Cada repository deve estender a interface `JpaRepository<Entity, Long>`.

Padrões do Repositório 

* Métodos de Consulta Personalizados 
* Siga as convenções de nomenclatura do Spring Data JPA 
- Exemplos: 
  - `findByTaskId(Long taskId)`
  - `findByStatusAndPriority(String status, String priority)`
* Quando usar @Query 
    - Consultas complexas com múltiplas junções 
    - Consultas nativas para operações específicas do banco de dados 
    - Projeções personalizadas 


## 4. Convenções de nomenclatura

- **Classes**: Devem ser nomeadas usando PascalCase (ex: `MinhaClasseController`, `MinhaClasseService`).
- **Métodos**: Devem ser nomeados usando camelCase (ex: `createProduto`, `getAllProdutos`).
- **Variáveis**: Devem ser nomeadas usando camelCase (ex: `produtoRepository`, `produtoList`).
- **Pacotes**: Devem ser nomeados usando letras minúsculas e separados por pontos (ex: `br.com.arcelino.produtosapi.controller`).


## 5. Registro de Logs 
- Para logs , use o SLF4J com a anotação `@Slf4j` 
- Exemplo: `log.info("Criando consulta para a produto: {}", produtoId);` 

Utilize níveis adequados:

```text
DEBUG → informações úteis para diagnóstico
INFO  → eventos relevantes da aplicação
WARN  → situações inesperadas que não interrompem a aplicação
ERROR → erros que precisam de atenção

## 6. Comentários 
- Javadoc em métodos de serviço públicos (opcional) 
- Comentários inline apenas para lógica complexa 
- O código deve ser autoexplicativo com nomes de métodos claros 


## 7. Instruções Específicas para Novas Funcionalidades 

Ao adicionar um novo recurso (ex.: Agendamentos): 

1. **Comece com a Entidade**: Crie a entidade JPA com todos os campos e relacionamentos necessários 
2. **Crie o Repositório**: Estenda o JpaRepository com métodos de consulta personalizados 
3. **Construa o Serviço**: Implemente a lógica de negócios com tratamento de erros adequado 
4. **Adicione o Controlador**: Crie endpoints REST seguindo o padrão 
5. **Atualize as Entidades Relacionadas**: Adicione relacionamentos às entidades existentes, se necessário.