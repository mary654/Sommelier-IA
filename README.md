# 🍷 Sommelier IA

Sistema web para adegas feito com **Spring Boot** e **Spring AI**. A IA lê o rótulo de uma bebida pela foto e cadastra a garrafa no banco de dados. Ela também sugere harmonizações para um prato, dando preferência ao que já está na adega, e escreve textos de vitrine e posts para redes sociais.

## Funcionalidades

| Tela | O que faz | Uso da GenAI |
|---|---|---|
| 📷 Ler rótulo | Envia a foto de uma garrafa e a bebida é cadastrada automaticamente | Visão: imagem → ficha estruturada (JSON) |
| 🍾 Minha adega | CRUD das bebidas: listar, buscar, cadastrar, editar e excluir | — |
| ✨ Vitrine | Gera e guarda textos de venda e posts com hashtags para cada bebida | Texto → texto estruturado |
| 🍽️ Harmonizar | Sugere 3 bebidas para um prato, priorizando as da adega, e guarda o histórico | Texto + dados do banco → texto estruturado |
| 🔊 Ouvir | Lê as respostas em voz alta | Leitura em voz alta do navegador |

## Tecnologias

- Java 21, Spring Boot 4.1.2
- Spring Web, Spring Data JPA, Spring Boot DevTools
- Spring AI 2.0.1 com o starter da OpenAI, apontado para a API do Google Gemini, que é compatível com o formato da OpenAI
- MySQL/MariaDB (XAMPP)
- HTML, CSS e JavaScript puro na interface

## Arquitetura

```
src/main/java/br/com/sommelier
├── controller   BebidaController, HarmonizacaoController, TratadorDeErros
├── service      SommelierService (IA), AdegaService, HarmonizacaoService
├── repository   interfaces Spring Data JPA
├── model        entidades JPA: Bebida, Vitrine, Harmonizacao, Sugestao
└── dto          formatos das respostas da IA
src/main/resources
├── application.properties
├── schema.sql   cria as tabelas
└── static       interface web
```

## Banco de dados

Quatro tabelas, criadas automaticamente na primeira execução:

- `bebidas`: as garrafas da adega
- `vitrines`: textos gerados pela IA para cada bebida (FK para `bebidas`)
- `harmonizacoes`: pratos consultados
- `sugestoes`: bebidas sugeridas em cada harmonização (FK para `harmonizacoes`)

## API

| Método | Caminho | Descrição |
|---|---|---|
| GET | `/api/bebidas?busca=` | lista ou busca bebidas |
| GET | `/api/bebidas/{id}` | detalha uma bebida |
| POST | `/api/bebidas` | cadastra manualmente |
| POST | `/api/bebidas/foto` | cadastra pela foto do rótulo (multipart, campo `foto`) |
| PUT | `/api/bebidas/{id}` | atualiza |
| DELETE | `/api/bebidas/{id}` | exclui |
| GET | `/api/bebidas/{id}/vitrines` | lista as vitrines da bebida |
| POST | `/api/bebidas/{id}/vitrines` | gera uma nova vitrine com IA |
| DELETE | `/api/bebidas/vitrines/{id}` | exclui uma vitrine |
| GET | `/api/harmonizacoes` | histórico |
| POST | `/api/harmonizacoes` | gera uma harmonização: `{"prato": "...", "ocasiao": "..."}` |
| DELETE | `/api/harmonizacoes/{id}` | exclui do histórico |

## Como rodar

### 1. Pré-requisitos

- Java 21 ou superior
- XAMPP com o MySQL ligado (usuário `root` sem senha, que é o padrão)
- Uma chave gratuita do Gemini, criada em https://aistudio.google.com/apikey

### 2. Informar a chave

A chave **não fica no código nem no GitHub**. Ela é lida da variável de ambiente `GEMINI_API_KEY`.

No PowerShell, dentro da pasta do projeto:

```powershell
$env:GEMINI_API_KEY="cole-a-chave-aqui"
```

A variável vale só para aquela janela do PowerShell.

### 3. Subir o sistema

```powershell
.\mvnw.cmd spring-boot:run
```

Quando aparecer `Started SommelierIaApplication`, abra http://localhost:8080.

O banco `sommelier` e as tabelas são criados sozinhos na primeira execução.

### Trocar o modelo ou o provedor

Tudo fica no `application.properties`:

```properties
spring.ai.openai.base-url=https://generativelanguage.googleapis.com/v1beta/openai
spring.ai.openai.chat.model=gemini-3.5-flash-lite
```

Para usar a OpenAI no lugar do Gemini, apague a linha do `base-url`, troque o modelo (por exemplo, `gpt-4o-mini`) e coloque a chave da OpenAI em `GEMINI_API_KEY`.
