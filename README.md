
import 'package:flutter/material.dart';
import 'package:shared_preferences/shared_preferences.dart';

void main() {
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return const MaterialApp(
      debugShowCheckedModeBanner: false,
      home: ExerciciosPage(),
    );
  }
}

class ExerciciosPage extends StatefulWidget {
  const ExerciciosPage({super.key});

  @override
  State<ExerciciosPage> createState() => _ExerciciosPageState();
}

class _ExerciciosPageState extends State<ExerciciosPage> {
  final TextEditingController tarefaController = TextEditingController();
  final TextEditingController nomeController = TextEditingController();
  final TextEditingController emailController = TextEditingController();

  List<String> tarefas = [];
  List<String> carrinho = [];

  final List<String> produtos = [
    'Produto A',
    'Produto B',
    'Produto C',
  ];

  String paginaSelecionada = 'Página A';

  @override
  void initState() {
    super.initState();
    carregarDados();
  }

  Future<void> carregarDados() async {
    final prefs = await SharedPreferences.getInstance();

    if (!mounted) return;

    setState(() {
      tarefas = prefs.getStringList('tarefas') ?? [];
      carrinho = prefs.getStringList('carrinho') ?? [];
      paginaSelecionada =
          prefs.getString('ultimaPagina') ?? 'Página A';

      nomeController.text = prefs.getString('nome') ?? '';
      emailController.text = prefs.getString('email') ?? '';
    });
  }

  // Exercício 1 - Limpar todas as tarefas
  Future<void> limparTarefas() async {
    final prefs = await SharedPreferences.getInstance();

    setState(() {
      tarefas.clear();
    });

    await prefs.setStringList('tarefas', tarefas);
  }

  Future<void> adicionarTarefa() async {
    final texto = tarefaController.text.trim();

    if (texto.isEmpty) return;

    final prefs = await SharedPreferences.getInstance();

    setState(() {
      tarefas.add(texto);
      tarefaController.clear();
    });

    await prefs.setStringList('tarefas', tarefas);
  }

  Future<void> removerTarefa(int index) async {
    final prefs = await SharedPreferences.getInstance();

    setState(() {
      tarefas.removeAt(index);
    });

    await prefs.setStringList('tarefas', tarefas);
  }

  // Exercício 3 - Última página acessada
  Future<void> selecionarPagina(String pagina) async {
    final prefs = await SharedPreferences.getInstance();

    setState(() {
      paginaSelecionada = pagina;
    });

    await prefs.setString('ultimaPagina', pagina);
  }

  // Exercício 4 - Formulário com salvamento automático
  Future<void> salvarCampo(String chave, String valor) async {
    final prefs = await SharedPreferences.getInstance();
    await prefs.setString(chave, valor);
  }

  // Exercício 5 - Carrinho persistente
  Future<void> adicionarCarrinho(String produto) async {
    final prefs = await SharedPreferences.getInstance();

    setState(() {
      carrinho.add(produto);
    });

    await prefs.setStringList('carrinho', carrinho);
  }

  @override
  void dispose() {
    tarefaController.dispose();
    nomeController.dispose();
    emailController.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Persistência de Dados'),
        backgroundColor: Colors.blue,
        foregroundColor: Colors.white,
      ),
      body: ListView(
        padding: const EdgeInsets.all(16),
        children: [
          const Text(
            'Exercícios 1 e 2 - Lista de Tarefas',
            style: TextStyle(
              fontSize: 20,
              fontWeight: FontWeight.bold,
            ),
          ),

          const SizedBox(height: 10),

          TextField(
            controller: tarefaController,
            decoration: const InputDecoration(
              labelText: 'Nova tarefa',
              border: OutlineInputBorder(),
            ),
          ),

          const SizedBox(height: 10),

          ElevatedButton(
            onPressed: adicionarTarefa,
            child: const Text('Adicionar tarefa'),
          ),

          ElevatedButton(
            onPressed: limparTarefas,
            child: const Text('Limpar tudo'),
          ),

          // Exercício 2 - Mensagem quando a lista estiver vazia
          if (tarefas.isEmpty)
            const Padding(
              padding: EdgeInsets.all(16),
              child: Text(
                'Nenhuma tarefa cadastrada.',
                textAlign: TextAlign.center,
              ),
            )
          else
            ...tarefas.asMap().entries.map((entrada) {
              return ListTile(
                title: Text(entrada.value),
                trailing: IconButton(
                  icon: const Icon(Icons.delete),
                  onPressed: () => removerTarefa(entrada.key),
                ),
              );
            }),

          const Divider(height: 40),

          const Text(
            'Exercício 3 - Última Página Acessada',
            style: TextStyle(
              fontSize: 20,
              fontWeight: FontWeight.bold,
            ),
          ),

          const SizedBox(height: 10),

          Wrap(
            spacing: 8,
            children: [
              ElevatedButton(
                onPressed: () => selecionarPagina('Página A'),
                child: const Text('Página A'),
              ),
              ElevatedButton(
                onPressed: () => selecionarPagina('Página B'),
                child: const Text('Página B'),
              ),
              ElevatedButton(
                onPressed: () => selecionarPagina('Página C'),
                child: const Text('Página C'),
              ),
            ],
          ),

          Text('Página selecionada: $paginaSelecionada'),

          const Divider(height: 40),

          const Text(
            'Exercício 4 - Formulário Persistente',
            style: TextStyle(
              fontSize: 20,
              fontWeight: FontWeight.bold,
            ),
          ),

          const SizedBox(height: 10),

          TextField(
            controller: nomeController,
            decoration: const InputDecoration(
              labelText: 'Nome',
              border: OutlineInputBorder(),
            ),
            onChanged: (valor) => salvarCampo('nome', valor),
          ),

          const SizedBox(height: 10),

          TextField(
            controller: emailController,
            keyboardType: TextInputType.emailAddress,
            decoration: const InputDecoration(
              labelText: 'Email',
              border: OutlineInputBorder(),
            ),
            onChanged: (valor) => salvarCampo('email', valor),
          ),

          const Divider(height: 40),

          const Text(
            'Exercício 5 - Carrinho de Compras',
            style: TextStyle(
              fontSize: 20,
              fontWeight: FontWeight.bold,
            ),
          ),

          const SizedBox(height: 10),

          ...produtos.map((produto) {
            return ListTile(
              title: Text(produto),
              trailing: ElevatedButton(
                onPressed: () => adicionarCarrinho(produto),
                child: const Text('Adicionar ao carrinho'),
              ),
            );
          }),

          const SizedBox(height: 20),

          const Text(
            'Itens no carrinho:',
            style: TextStyle(fontWeight: FontWeight.bold),
          ),

          if (carrinho.isEmpty)
            const Text('Carrinho vazio.')
          else
            ...carrinho.map((produto) {
              return ListTile(
                leading: const Icon(Icons.shopping_cart),
                title: Text(produto),
              );
            }),
        ],
      ),
    );
  }
}
