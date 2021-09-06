# Trabalho 01 - Princípio da Responsabilidade Única
- Universidade Federal de São Carlos - Departamento de Computação
- Programação Orientada a Objetos Avançada
- Aluno: Roseval Donisete Malaquias Junior - RA: 758597
- Professor: Delano Medeiros Beder

---

## Introdução
O desenvolvimento de softwares orientados a objetos em larga escala apresenta dificuldades devido a alta complexidade de organização desta atividade. Diante disso, Robert C. Martin (Uncle Bob) buscou elicitar um conjunto de princípios a serem seguidos na arquiteturação de softwares orientados a objetos a fim de facilitar o entendimento e desenvolvimento destes programas. Com isso, surgiu-se o termo SOLID como um acrônimo dos cinco princípios de design de programação, destinados ao desenvolvimento de softwares com maior flexibilidade de extensão e manutenção. Neste artigo, será feita a apresentação do **[S]** *Single Resposability Principle* (SRP) como o primeiro princípio SOLID por meio da explicação conceitual e exemplificação de sua aplicação.

## Princípio da Responsabilidade Única
O SRP reza que todo módulo de código desenvolvido deve possuir apenas uma responsabilidade com relação às funcionalidades do programa, apresentando assim apenas um eixo de mudança. Os programas desenvolvidos com esse princípio apresentam alta coesão e baixo acoplamento dado a especificidade dos blocos de código produzidos. Diante disso, a manutenção destes programas será facilitada devido a baixa interdependência e focalização da responsabilidade de cada bloco. Esses módulos podem ser melhor reutilizados com menores chances de encadeamento de erros, visto que o programa estará melhor organizado em módulos bem definidos.

Entretanto, a aplicação deste princípio pode ser complicada, sendo muito dependente da experiência do programador para a detecção de responsabilidades no âmbito do programa em desenvolvimento. Como por exemplo, um programador pouco experiente que está trabalhando com um produto a não muito tempo, não tem em mente ainda as necessidades e características deste produto. Isso pode acarretar o desenvolvimento de classes com múltiplas responsabilidades, como pode ser observado no exemplo apresentado a seguir da classe ``EmailService``, encarregada de enviar emails para clientes de um sistema web de compras online.

### Quebra do Princípio da Responsabilidade Ùnica
Diante do código apresentado a seguir, é possível identificar que a classe ``EmailService`` possui mais de uma responsabilidade, ferindo então o SRP. Essa classe está encarregada de autenticar o email da empresa para realizar o envio do email ao cliente, e efetuar a conexão e consulta ao banco de dados do sistema web a fim de recuperar o email do cliente desejado. 

```Java
//...
// Classe que envia um email de confirmação de compra ao cliente.
public class EmailService {

	public void send(String empresaEmail, String clienteID) {

		try {
			// Realiza a conexao ao sql server
			Class.forName("com.mysql.cj.jdbc.Driver");
			Connection conexao = DriverManager.getConnection("jdbc:mysql://localhost:3306/SISTEMACOMPRAS", "root", "root");

			// Realiza a consulta do email do cliente 
			String consultaSql = "SELECT * FROM CLIENTE WHERE ID = ?";
			String emailCliente = null;
			PreparedStatement stat = conexao.prepareStatement(consultaSql);

			stat.setString(1, clienteID);
			ResultSet resultados = stat.executeQuery();
			if (resultados.next()) {
				emailCliente = resultados.getString("email");
			}

			resultados.close();
			stat.close();
			conexao.close();

			// Carregando o arquivo de propriedades do projeto
			Properties propriedades = new Properties();
			InputStream  streamPropriedades = EmpresaController.class.getClassLoader().
			    getResourceAsStream("config.properties");

			if (streamPropriedades != null) {
				propriedades.load(streamPropriedades);
			} else {
				throw new FileNotFoundException("config.properties não encontrado!");
			}

			// Inicializa sessão com email da empresa para o envio
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
			message.setFrom(new InternetAddress("empresa@email.com")); // empresa envia o email
			message.setRecipient(Message.RecipientType.TO, new InternetAddress(emailCliente)); // cliente recebe o email
			message.setSubject("Compra efetuada");

			MimeBodyPart mimeBodyPart = new MimeBodyPart();
			mimeBodyPart.setContent("Olá,\n" + "Sua compra foi efetuada com sucesso!", "text/plain");

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

Esses indícios da presença de múltiplas responsabilidades na classe ``EmailService`` podem ser observados também na utilização do serviço disponibilizado por essa classe, visto que o email do cliente não é passado como entrada na chamada do método ```send```, mas sim o ID do cliente para que o email deste cliente seja retornado com um acesso e consulta ao banco de dados dentro da classe. Portanto, além dessa classe ser responsável por enviar o email, ela também precisa manusear o banco de dados, o que demonstra a presença de múltiplas responsabilidades em uma única classe. A seguir é feita a demonstração da utilização do serviço do envio de email de confirmação da compra do cliente por meio da classe ``Principal``.


```Java
// ...
// Demonstracao da utilizacao do servico de envio de email ao cliente.
public class Principal {

	public static void main(String[] args){

		// ...
		// Tratando como se ja tivesse identificado o id (clienteId) do cliente que será enviado o email.
		EmailService email = new EmailService();
		email.send("empresa@email.com", clienteId);
	}
}
```

Tendo em vista que, as responsabilidades de um bloco de código estão atreladas aos papéis que os programadores desempenham no desenvolvimento de software. A classe ``EmailService`` é mantida por dois grupos diferentes de programadores encarregados deste sistema web, o time de marketing que administra o envio de emails e o time de banco de dados que administra o banco e toda sua arquitetura. A manutenção desta classe com múltiplas responsabilidades por dois times diferentes pode desencadear erros não esperados. Portanto, é desejado que as múltiplas responsabilidades desta classe sejam quebradas em múltiplos módulos de código para respeitar o SRP, como demonstrado a seguir.

### Conexão e Consulta ao Banco de Dados
A fim de isolar a responsabilidade de conexão e consulta ao banco de dados do sistema, foi utilizado o princípio de Objeto de Acesso a Dados (DAO), que implica na separação das regras de negócio das regras de acesso ao banco de dados. Assim, com o isolamento desta responsabilidade, foi produzida a classe ``Cliente`` para encapsular os dados da tabela cliente e a classe ``ClienteDAO`` que realiza a conexão ao banco de dados e a atualização, exclusão e inserção de tuplas à tabela cliente.

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
// Essa classe esta responsavel pelo CRUD da tabela Cliente.
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
	// Read
	public Cliente busca(String id) {
		Cliente cliente = null;

		String consultaSql = "SELECT * FROM CLIENTE WHERE ID = ?";
		String nome = null, cpf = null, email = null, senha = null, 

		try {
			Connection conexao = this.getConnection();
			PreparedStatement stream = conexao.prepareStatement(consultaAql);

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

// ...

}
```

### Envio do Email ao Cliente
Agora, é possível isolar a responsabilidade de envio de emails aos clientes. Sendo assim, foi produzida a classe ``EmailService`` que realiza a autenticação da conta de email da empresa, e envia um email de confirmação de compra ao email especificado. Além disso, agora o método ``send`` não mais recebe o ID do cliente, mas sim o seu email, para que essa classe não tenha conhecimento de como funciona a consulta a cliente que foge do escopo especificado à essa classe.

```Java
// ...
// Classe que envia um email de confirmação de compra ao cliente.
public class EmailService {

	public void send(String empresaEmail, String clienteEmail) {

		try {
			// Carregando o arquivo de propriedades do projeto
			Properties propriedades = new Properties();
			InputStream  streamPropriedades = EmpresaController.class.getClassLoader().
			    getResourceAsStream("config.properties");

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
			message.setFrom(new InternetAddress("empresaEmail")); // empresa envia o email
			message.setRecipient(Message.RecipientType.TO, new InternetAddress("clienteEmail")); // cliente recebe o email
			message.setSubject("Compra efetuada");

			MimeBodyPart mimeBodyPart = new MimeBodyPart();
			mimeBodyPart.setContent("Olá,\n" + "Sua compra foi efetuada com sucesso!", "text/plain");

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

### Utilização do Serviço
Por fim, agora a classe ``EmailService`` está respeitando o SRP, sendo assim, esse serviço de envio de email para a confirmação de compra pode ser reutilizado dentro do sistema web ou em outros projetos caso desejado. Nesta classe ``Principal`` é exemplificado a utilização deste serviço. Inicialmente, é feito a chamada do método ``ClienteDAO.buscar(clienteId)`` para obter as informações do cliente, dentre elas o email desejado, e a chamada do método ``EmailService.send("empresa@email.com", cliente.getEmail())`` para efetuar o envio do email de confirmação da compra.

```Java
// ...
// Demonstracao da utilizacao do servico de envio de email ao cliente.
public class Principal {

	public static void main(String[] args){

		// ...
		// Tratando como se ja tivesse identificado o id (clienteId) do cliente que será enviado o email.
		ClienteDAO dao = new ClienteDAO();
		EmailService emailSender = new EmailService();
		Cliente cliente = dao.buscar(clienteId);
		emailSender.send("empresa@email.com", cliente.getEmail());
	}
}
```
## Conclusão
Através da análise inicial da classe ``EmailService``, foram identificadas as múltiplas responsabilidades ligadas a essa classe, e assim também, os múltiplos times de desenvolvimento que estariam encarregados de realizar a manutenção desta classe. Diante disso, mostrou-se necessário a refatoração da classe a fim de evitar a ocorrência de erros inesperados no sistema, aplicando o SRP. Importante destacar que, o SRP é um **princípio** e não uma lei no desenvolvimento de software, entretanto, a sua não utilização em um projeto deve ser devidamente justificada e elicitada no documento de requisitos do projeto. Por fim, os dois times de desenvolvimento estão responsáveis por partes distintas do sistema, sendo que cada classe apresentada tem uma única responsabilidade.
