# To-Do List - FIAP

Este é um aplicativo Android de gerenciamento de tarefas (To-Do List) desenvolvido como parte da atividade individual da FIAP. O objetivo principal é implementar a camada de UI, navegação e integração com uma arquitetura de persistência local já existente.

## Tecnologias Utilizadas
* **Kotlin:** Linguagem de programação principal.
* **Jetpack Compose:** Toolkit moderno e declarativo para construção da interface de usuário (UI).
* **Room:** Biblioteca de persistência para abstração do banco de dados SQLite.
* **Coroutines e Flow:** Gerenciamento de operações assíncronas e reatividade de dados.
* **ViewModel:** Gerenciamento de estado da UI com ciclo de vida consciente.
* **Navigation Compose:** Navegação fluida entre telas no ecossistema Compose.

## Arquitetura e Implementação

### 1. TarefaRepository
O `TarefaRepository` atua como a única fonte de verdade (Single Source of Truth) para os dados. Ele encapsula a lógica de acesso a dados do `TarefaDao`, isolando a `ViewModel` das complexidades do banco de dados Room. Suas operações incluem buscar fluxos de dados, salvar, atualizar status e excluir tarefas.

### 2. TarefaViewModel
A `TarefaViewModel` gerencia o estado da interface. Ela consome os dados do `TarefaRepository` convertendo-os em um `StateFlow` observável pela UI. Utilizando a `viewModelScope`, ela lança corrotinas para executar operações de banco de dados (inserir, atualizar, deletar) de forma assíncrona, sem bloquear a thread principal.

### 3. ListaTarefasScreen
Esta tela exibe as tarefas utilizando uma `LazyColumn` para alta performance. Ela não retém estado próprio de dados; em vez disso, observa o `StateFlow` da `ViewModel`. As ações do usuário (clicar para editar, concluir ou excluir) são repassadas para cima (State Hoisting) via callbacks, mantendo o componente desacoplado e testável.

### 4. FormularioTarefaScreen
A tela de formulário é inteligente e reaproveitável. Ela diferencia o modo de **cadastro** e **edição** através da presença ou ausência da variável `tarefaExistente`. Se uma tarefa for recebida, os campos de texto (`titulo` e `descricao`) são inicializados com os valores atuais. Caso contrário, iniciam em branco para uma nova entrada.

### 5. AppNavigation e Rotas
O `AppNavigation` define o `NavHost` da aplicação. A rota do formulário é configurada para receber um argumento opcional: o ID da tarefa (`tarefaId`). Quando o usuário clica em "Nova Tarefa", é passado o ID `-1`, indicando criação. Quando clica em editar, o ID real é passado, permitindo que a tela recupere a tarefa correta.

### 6. MainActivity
A `MainActivity` serve como ponto de entrada. Sua responsabilidade foi reduzida à inicialização do banco de dados `AppDatabase`, criação da `TarefaViewModel` utilizando sua `Factory` para injeção do repositório, e inicialização da árvore de componentes Compose através do `AppNavigation`.