# Trabalho 01 - Princípio da Responsabilidade Única
- Universidade Federal de São Carlos - Departamento de Computação
- Programação Orientada a Objetos Avançada
- Aluno: Roseval Donisete Malaquias Junior - RA: 758597
- Professor: Delano Medeiros Beder

## Introdução
O desenvolvimento de softwares orientados em larga escla apresenta dificuldades intrínsecas à alta complexidade desta atividade. Diante disso, Robert C. Martin (Uncle Bob) buscou elicitar um conjunto de postulados a serem seguidos na arquiteturação de softwares orientados a objetos a fim de facilitar o entendimento e desenvolvimento destes programas. Com isso, surgiu-se o termo SOLID como um acrônimo dos cinco postulados de design de programação, destinados ao desenvolvimento de softwares com maior flexibilidade de extensão e manutenção. Neste artigo, será feita a apresentação do **[S]** *Single Resposability Principle* (SRP) como o primeiro principio SOLID por meio da explicação conceitual do postulado e exemplificação de sua aplicação.

## Princípio da Resposabilidade Única
O SRP reza que todo módulo de código desenvolvido deve possuir apenas uma responsabilidade com relação as funcionalidade do programa, apresentando assim apenas um eixo de mudança. Os programas desenvolvidas com esse principio apresentam alta coesão e baixo acoplamento dado a especificidade dos blocos de código produzidos. Sendo assim, a manutenção destes programas será facilitados devido a baixa interdependência entre os diversos blocos de código e focalização das responsabilidade de cada bloco. Esses módulos podem ser melhor reutilizados com menores chances de encadeamento de erros, visto que o programa estará melhor organizado em módulos bem definidos.

Entretanto, a aplicação deste postulado pode ser complicada, sendo muito dependente da experiencia do programador para a detecção de responsabilidades no âmbito do programa em desenvolvimento. Como por exemplo, muitas vezes para duas pessoas diferentes, um mesmo bloco de código pode possuir um numero diferente de responsabilidades, sendo que as experiencias dessas duas pessoas na área podem ser diferentes, produzindo visões duas de uma mesma solução. Isso pode ocasionar o desenvolvimento de classes com multiplas resposanbilidades, como pode ser observado na classe ``EmailService``, encarregada de enviar um email para clientes de um sistema web de compras online.

```Java
//...
// Classe que envia um email da compra de um cliente para ele.
public class EmailService {

	public void send(InternetAddress empresaEmail, String clienteID) {

		try {

			// Carregando o arquivo de propriedades do projeto
			Properties propriedades = new Properties();
			InputStream  streamPropriedades = EmpresaController.class.getClassLoader().getResourceAsStream("config.properties");

			if (streamPropriedades != null) {
				propriedades.load(streamPropriedades);
			} else {
				throw new FileNotFoundException("config.properties não encontrado!");
			}

			// Inicializa sessao com email da empresa para o envio
			String login = propriedades.getProperty("login");
			String senha = propriedades.getProperty("senha");

			Session session = Session.getInstance(propriedades, new Authenticator() {
				@Override
				protected PasswordAuthentication getPasswordAuthentication() {
					return new PasswordAuthentication(login, senha);
				}
			});

			// Realiza a conexao ao sql server.
			try {
				Class.forName("com.mysql.cj.jdbc.Driver");
			} catch (ClassNotFoundException e) {
				throw new RuntimeException(e);
			}

			// Realiza a consulta das informacoes necessarias do cliente (email e nome) 
			String consultaSql = "SELECT * FROM CLIENTE WHERE ID = ?";
			String nomeCliente = null, emailCliente = null;

			try {
				Connection conexao = this.getConnection();
				PreparedStatement stat = conexao.prepareStatement(consultaSql);

				stat.setLong(1, clienteID);
				ResultSet resultados = stat.executeQuery();
				if (resultados.next()) {
					nomeCliente = resultados.getString("nome");
					emailCliente = resultados.getString("email");
				}

				resultados.close();
				stat.close();
				conexao.close();
			} catch (SQLException e) {
				throw new RuntimeException(e);
			}

			// Inicializando o envio do email 
			Message message = new MimeMessage(session);
			message.setFrom("empresa@email.com"); // empresa envia o email
			message.setRecipient(Message.RecipientType.TO, emailCliente); // cliente recebe o email
			message.setSubject("Compra efetuada");

			MimeBodyPart mimeBodyPart = new MimeBodyPart();
			mimeBodyPart.setContent("Olá " + nomeCliente + ", compra efetuada com sucesso!", "text/plain");

			Multipart multipart = new MimeMultipart();
			multipart.addBodyPart(mimeBodyPart);

			message.setContent(multipart);
			Transport.send(message);

			System.out.println("Mensagem enviada!");

		} catch (Exception e) {
			System.out.println("Mensagem não enviada!");
			e.printStackTrace();
		}
	}
}
```
Diante do código apresentado é possível identificar que a classe ``EmailService`` tem mais de uma responsabilidade. Além da classe estar encarregada de realizar a configuração da autenricação do email da empresa através da classe ``Properties`` para realizar o envio do email. Essa classe também realiza a conexão, acesso e consulta ao banco de dados do sistema web para recuperar o email do cliente desejado.

Sendo que, as responsabilidades de um bloco de código esta atrelada aos papeis que as pessoas desempenham no desenvolvimento de software. A classe apresentada é manuseada por dois grupos diferentes de pessoas encarregadas deste sistema web, o time de marketing que administra o envio de emails e o time de banco de dados que administra toda a arquitetura do banco. O manuseio desta classe com multiplas responsabilidades por dois times diferentes durante o desenvolvimento deste sistema pode ocasionar em varios encadeamentos de bug não esperados. Portanto, é desejado que as multiplas responsabilidades desta classe sejam quebradas em multiplos módulos de código, como demostrado a seguir.

## Acesso, Conexão e Consulta ao Banco de Dados
A fim de isolar a manuseio do BD da classe ``EmailReceiver``, foi utilizado o principio de DAO, que implica na utilização de uma classe para armazenar as informaçõe de uma tabela e outra classe DAO que realização a conexão e realiza o update, delete e insert na tabela. Diante disso, tem-se o seguinte código resultante da classe ``EmailReceier``, ``Cliente`` e ``ClienteDAO``.

```Java
// ...
// Classe Domain para cliente que armazena os atributos da tabela Cliente.
public class Cliente {

	private String id;
	private String cpf;
	private String nome;
	private String senha;
	private String email;

	public Cliente(String id, String cpf, String nome, String senha, String email) {
		this.id = id;
		this.cpf = cpf;
		this.nome = nome;
		this.senha = senha;
		this.email = email;
	}

	public String getId() {
		return id;
	}
	//...
	
}
```
```Java
// ...

// Essa classe esta responsavel pelo CRUD da classe Cliente.
public class ClienteDAO {

	// Construtor que apenas realiza a conexao ao driver jdbc
	public ClienteDAO() {
		try {
			Class.forName("com.mysql.cj.jdbc.Driver");
		} catch (ClassNotFoundException e) {
			throw new RuntimeException(e);
		}
	}

	// Efetua a conexao ao sql server
	protected Connection getConnection() throws SQLException {
		return DriverManager.getConnection("jdbc:mysql://localhost:3306/SISTEMACOMPRAS", "root", "root");
	}

	// Implementacao do CRUD...
	// Create
	public void insere(Cliente cliente) {
		
		String consultaSql = "INSERT INTO CLIENTE (ID, NOME, CPF, EMAIL, SENHA) VALUES(?, ?, ?, ?, ?)";

		try {
			Connection conexao = this.getConnection();
			PreparedStatement stream = conexao.prepareStatement(consultaSql, Statement.RETURN_GENERATED_KEYS);

			stream = conexao.prepareStatement(sql);
			stream.setString(1, cliente.getId());
			stream.setString(2, cliente.getNome());
			stream.setString(2, cliente.getCpf());
			stream.setString(3, cliente.getEmail());
			stream.setString(4, cliente.getSenha());
			stream.executeUpdate();

			stream.close();
			conexao.close();
		} catch (SQLException e) {
			throw new RuntimeException(e);
		}
	}

	// Delete
	public void deleta(Profissional profissional) {
		String consultaSql = "DELETE FROM CLIENTE WHERE ID = ?";

		try {
			Connection conexao  = this.getConnection();
			PreparedStatement stream  = conexao.prepareStatement(consultaSql);

			stream.setString(1, cliente.getId());
			stream.executeUpdate();

			stream.close();
			conexao.close();
		} catch (SQLException e) {
			throw new RuntimeException(e);
		}
	}

	// Update
	public void atualiza(Profissional profissional) {

		String consultaSql = "UPDATE CLIENTE SET NOME = ?, CPF = ?, EMAIL = ?, SENHA = ? WHERE ID = ?";

		try {
			Connection conexao = this.getConnection();
			PreparedStatement stream = conexao.prepareStatement(consultaSql);

			stream.setString(1, cliente.getNome());
			stream.setString(2, cliente.getCpf());
			stream.setString(3, cliente.getEmail());
			stream.setLong(4, cliente.getSenha());
			stream.setLong(5, cliente.getId());
			stream.executeUpdate();

			stream.close();
			conexao.close();
		} catch (SQLException e) {
			throw new RuntimeException(e);
		}
	}

	// Read
	public Cliente busca(String id) {
		Cliente cliente = null;

		String consultaSql = "SELECT * FROM CLIENTE WHERE ID = ?";
		String nome = null, cpf = null, email = null, senha = null, 

		try {
			Connection conexao = this.getConnection();
			PreparedStatement stream = conexao.prepareStatement(sql);

			stream.setString(1, id);
			ResultSet resultados = stream.executeQuery();
			if (resultados.next()) {
				nome = resultados.getString("nome");
				cpf = resultados.getString("cpf");
				email = resultados.getString("email");
				senha = resultados.getString("senha");
			}

			resultados.close();
			stream.close();
			conexao.close();
		} catch (SQLException e) {
			throw new RuntimeException(e);
		}
	}
}
```

## Envio do Email ao Cliente
A classe ``EmailSender`` tem a resposabilidade de realizar a autenticação a conta de email da empresa, e enviar o email desejado ao email recebido como entrada do método ``send``. Além disso, agora o método ``send`` não mais recebe o ID do cliente, mas sim o email para que essa classe não tenha conhecimento de como funciona a consulta ao cliente que não pertece ao seu escopo.

```Java
// ...

// Classe que envia um email da compra do cliente para ele.
public class EmailService {


	public void send(InternetAddress empresaEmail, InternetAddress clienteEmail) {

		try {

			// Carregando o arquivo de propriedades do projeto
			Properties propriedades = new Properties();
			InputStream  streamPropriedades = EmpresaController.class.getClassLoader().getResourceAsStream("config.properties");

			if (streamPropriedades != null) {
				propriedades.load(streamPropriedades);
			} else {
				throw new FileNotFoundException("config.properties não encontrado!");
			}

			// Inicializa sessao com email da empresa para o envio
			String login = propriedades.getProperty("login");
			String senha = propriedades.getProperty("senha");

			Session session = Session.getInstance(propriedades, new Authenticator() {
				@Override
				protected PasswordAuthentication getPasswordAuthentication() {
					return new PasswordAuthentication(login, senha);
				}
			});

			// Inicializando o envio do email 
			Message message = new MimeMessage(session);
			message.setFrom(empresaEmail); // empresa envia o email
			message.setRecipient(Message.RecipientType.TO, clienteEmail); // cliente recebe o email
			message.setSubject("Compra efetuada");

			MimeBodyPart mimeBodyPart = new MimeBodyPart();
			mimeBodyPart.setContent("Olá " + nomeCliente + ", sua compra nesta loja foi efetuada com sucesso!", "text/plain");

			Multipart multipart = new MimeMultipart();
			multipart.addBodyPart(mimeBodyPart);

			message.setContent(multipart);
			Transport.send(message);

			System.out.println("Mensagem enviada!");

		} catch (Exception e) {
			System.out.println("Mensagem não enviada!");
			e.printStackTrace();
		}
	}
}
```

## Classe Principal
Com isso, a classe pricipal fica responsavel por realizar a chamada do ``ClientDAO`` para retornar o email do cliente e realizar a instanciação da classe ``EmailSender`` e a chamada do método ``send(email)`` para efetuar o envio do email.

```Java

// ...

// Classe que envia um email da compra do cliente para ele.
public class Principal {

	ClienteDAO dao = new ClienteDAO();
	EmailService emailSender = new EmailService();

	public static void main(String[] args){

		// ...
		// Tratando como se ja tivesse identificado o id do cliente que será enviado o email.
		
		Cliente cliente = dao.buscar(clienteId);
		emailSender.send(new InternetAddress("empresa@email.com"), new InternetAddress(cliente.getEmail));
	}
}

```

Através da análise inicial da classe ``EmailSender``, foram identificas as múltiplas responsabilidades ligadas a essa classe, e assim também os múltiplos times de desenvolvimento que estariam encarregados de realizar a manutenção desta única classe. Diante disso, foi identificado a necessidade de refatoração da classe a fim de evitar a ocorrência de erros inesperados no sistema. Importante destacar que, o SRP é um *principio* e não uma lei no desenvolvimento de software, entretanto, a sua não utilização em um projeto deve ser devidamente justificada e elicitada no documento de requisitos do projeto. Por fim, os dois times de desenvolvimento estão responsaveis por partes distitntas do sistema, sendo cada classe apresentada tém uma das responsabilidade geradas pela realização desta tarefa.

## Conclusão -> Talvez não
