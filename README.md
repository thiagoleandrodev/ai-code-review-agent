# 🤖 Copilot Code Review Agent

Agente inteligente para revisão de qualidade de código desenvolvido com **Microsoft Copilot Studio**, capaz de analisar trechos de código, sugerir melhorias e aplicar boas práticas automaticamente.

---

## 📌 Sobre o Projeto

Este projeto tem como objetivo criar um agente de IA especializado em **análise de qualidade de código**, utilizando os recursos do **Microsoft Copilot Studio**.

O agente atua como um revisor automatizado, ajudando desenvolvedores a:

* Identificar problemas no código
* Melhorar legibilidade e organização
* Aplicar boas práticas de desenvolvimento
* Reduzir bugs e inconsistências

---

## 🚀 Funcionalidades

* 🔍 Análise automática de código
* 💡 Sugestões de melhorias
* 📏 Avaliação de boas práticas
* 🧠 Uso de IA para revisão contextual
* ⚡ Respostas rápidas e automatizadas

---

## 🛠️ Tecnologias Utilizadas

* Microsoft Copilot Studio
* Inteligência Artificial
* Python

---

## 🧩 Arquitetura do Projeto

O agente foi projetado com base em:

* Entrada de código fornecida pelo usuário
* Processamento via IA no Copilot Studio
* Geração de feedback estruturado
* Retorno com sugestões de melhoria

---

## ▶️ Como Executar

### 1. Clone o repositório

```bash
git clone https://github.com/seu-usuario/copilot-code-review-agent.git
```

### 2. Acesse a pasta do projeto

```bash
cd copilot-code-review-agent
```

### 3. Configure o ambiente

* Configure suas credenciais do Microsoft Copilot Studio
* Um exemplo base de codigo:
  Você é um revisor de código sênior chamado Thiago Leandro Dos Santos. Revise diffs ou arquivos de código com foco em:
- Corrija e casos extremos
- Segurança (segredos, injeções, traversal de caminho, desserialização insegura, SSRF, SCARF, XSS, CSRF)
- Desempenho e escalabilidade
- Confiabilidade (exceções, vazamento de recursos)
- Manutenibilidade (nomes, estrutura, testes)
- Boas práticas de Cloud/Azure DevOps, quando relevante
Return a STRICT JSON with:
{{
  "file": "string",
  "issues":
      {{
        "title":  "short",
        "severity": "critical | high | medium | low",
        "line": int | null,
        "description": "O que está errado",
        "recommendation": "Como corrigir",
        "tags": ["security" | "performace" | "readability" | ...]
      }}
  ]
}}
Se não houver problemas: return {{"fire":"...", "issues":[]}}.
Nunca inclua code fences em Markdown. Não escreva texto fora do JSON.

* Ajuste variáveis de ambiente (se necessário)

### 4. Execute o projeto

(Specifique aqui como rodar — ex: script, interface, etc.)

---

## 📷 Exemplo de Uso

Entrada:
```txt
import sqlite3
def buscar_usuario(nome):
conn = sqlite3.connect("banco.db")
cursor = conn.cursor()
# VULNERÁVEL: concatenação direta de strings
cursor.execute(f"SELECT * FROM usuarios WHERE nome = '{nome}'")
return cursor.fetchall()

```
Saída:
```txt

{
  "file": "buscar_usuario.py",
  "issues": [
    {
      "title": "Vulnerabilidade crítica de SQL Injection",
      "severity": "critical",
      "line": 5,
      "description": "A concatenação direta do parâmetro 'nome' na query SQL permite SQL Injection. Um atacante pode inserir: nome = \"' OR '1'='1\" para retornar todos os usuários, ou \"'; DROP TABLE usuarios; --\" para apagar a tabela. Classificado como OWASP Top 10 A03:2021 - Injection.",
      "recommendation": "NUNCA concatenar strings em SQL. Usar queries parametrizadas: cursor.execute('SELECT * FROM usuarios WHERE nome = ?', (nome,)). O driver sqlite3 automaticamente escapa valores perigosos.",
      "tags": ["security", "sql-injection", "critical", "owasp-top10"]
    },
    {
      "title": "Vazamento de recursos (resource leak)",
      "severity": "high",
      "line": 3,
      "description": "A conexão ao banco de dados e o cursor não são fechados após uso. Em aplicações com múltiplas requisições, isso causa esgotamento de conexões, file descriptors e potencial travamento da aplicação.",
      "recommendation": "Usar context manager: with sqlite3.connect('banco.db') as conn: with conn.cursor() as cursor: ... Ou chamar explicitamente cursor.close() e conn.close() em bloco finally.",
      "tags": ["reliability", "resource-management", "memory-leak"]
    },
    {
      "title": "Ausência de tratamento de exceções",
      "severity": "high",
      "line": 2,
      "description": "Não há tratamento para erros de banco (sqlite3.Error, OperationalError, DatabaseError). Se o arquivo não existir, houver corrupção ou erro de sintaxe SQL, a exceção propaga sem contexto adequado.",
      "recommendation": "Envolver em try-except: try: ... except sqlite3.Error as e: logger.error(f'Erro ao buscar usuário: {e}'); raise CustomDatabaseException('Erro ao consultar banco de dados') from e",
      "tags": ["reliability", "error-handling", "exception-handling"]
    },
    {
      "title": "Caminho do banco de dados hardcoded",
      "severity": "medium",
      "line": 3,
      "description": "O caminho 'banco.db' está hardcoded no código. Isso dificulta testes (não pode mockar), configuração entre ambientes (dev/prod) e portabilidade da aplicação.",
      "recommendation": "Passar caminho como parâmetro com valor padrão, ou carregar de variável de ambiente/config: db_path = os.getenv('DB_PATH', 'banco.db'). Melhor ainda: usar padrão Repository/DAO com injeção de dependência.",
      "tags": ["maintainability", "testability", "configuration"]
    },
    {
      "title": "Retorna dados brutos sem validação",
      "severity": "medium",
      "line": 7,
      "description": "fetchall() retorna lista de tuplas brutas sem validação, tipo ou estrutura definida. Isso dificulta manutenção, pode expor dados sensíveis não intencionais e viola princípios de encapsulamento.",
      "recommendation": "Retornar objetos tipados (dataclass, NamedTuple ou Pydantic model). Exemplo: return [Usuario(*row) for row in cursor.fetchall()]. Implementar DTOs para expor apenas campos necessários.",
      "tags": ["maintainability", "data-validation", "security"]
    },
    {
      "title": "Falta de validação e sanitização de entrada",
      "severity": "medium",
      "line": 2,
      "description": "O parâmetro 'nome' não é validado. Aceita None, strings vazias, tipos inválidos ou payloads maliciosos. Mesmo com queries parametrizadas, validação de entrada é defesa em profundidade.",
      "recommendation": "Validar entrada: if not nome or not isinstance(nome, str): raise ValueError('Nome deve ser string não vazia'). Considerar também validar tamanho máximo e caracteres permitidos.",
      "tags": ["input-validation", "security", "defense-in-depth"]
    },
    {
      "title": "Ausência de type hints",
      "severity": "low",
      "line": 2,
      "description": "A função não possui anotações de tipo, dificultando compreensão, manutenção e detecção de erros em tempo de desenvolvimento com ferramentas como mypy.",
      "recommendation": "Adicionar type hints: def buscar_usuario(nome: str) -> list[tuple]: ... Melhor ainda: criar modelo tipado e retornar list[Usuario].",
      "tags": ["maintainability", "type-safety", "readability"]
    },
    {
      "title": "Falta de documentação (docstring)",
      "severity": "low",
      "line": 2,
      "description": "Ausência de docstring explicando propósito, parâmetros, retorno, exceções possíveis e comportamento esperado da função.",
      "recommendation": "Adicionar docstring completa seguindo padrão: '''Busca usuários no banco de dados pelo nome.\\n\\nArgs:\\n    nome (str): Nome do usuário\\n\\nReturns:\\n    list[tuple]: Lista de registros encontrados\\n\\nRaises:\\n    ValueError: Se nome for inválido'''",
      "tags": ["documentation", "maintainability"]
    },
    {
      "title": "Ausência de logging e auditoria",
      "severity": "medium",
      "line": null,
      "description": "Não há registro de tentativas de acesso ao banco. Para segurança e debugging, é importante logar operações de leitura, especialmente com dados sensíveis de usuários.",
      "recommendation": "Adicionar logging: import logging; logger.info(f'Buscando usuário: {nome[:50]}'); logger.debug(f'Query executada: {query}'). Considerar auditoria para compliance (LGPD/GDPR).",
      "tags": ["security", "observability", "compliance", "lgpd"]
    },
    {
      "title": "Potencial Path Traversal se nome do DB vier de entrada",
      "severity": "low",
      "line": 3,
      "description": "Embora neste caso o path seja hardcoded, se futuramente o nome do banco vier de entrada do usuário, há risco de path traversal (../../../etc/passwd).",
      "recommendation": "Se permitir paths configuráveis, validar: usar os.path.abspath(), verificar que está em diretório permitido, sanitizar com pathlib.Path().resolve() e validar contra whitelist.",
      "tags": ["security", "path-traversal", "future-proofing"]
    }
  ]
}

```

---

## 📈 Possíveis Melhorias

* Integração com CI/CD
* Suporte a múltiplas linguagens
* Interface web para interação
* Histórico de análises

---

## 🤝 Contribuição

Contribuições são bem-vindas!

1. Faça um fork do projeto
2. Crie uma branch (`git checkout -b feature/minha-feature`)
3. Commit suas mudanças (`git commit -m 'Minha contribuição'`)
4. Push (`git push origin minha-feature`)
5. Abra um Pull Request

---

## 📄 Licença

Este projeto está sob a licença MIT.
Sinta-se livre para usar e modificar.

---

## 👨‍💻 Autor

Desenvolvido por você 🚀
(Adicione seu nome e LinkedIn aqui)

---

## ⭐ Considerações Finais

Este projeto demonstra o uso de IA aplicada à engenharia de software, com foco em automação e melhoria contínua da qualidade de código.

Se você gostou do projeto, deixe uma ⭐ no repositório!
