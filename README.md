# Trabalho 01 - Princípio da Responsabilidade Única
- Universidade Federal de São Carlos - Departamento de Computação
- Programação Orientada a Objetos Avançada
- Aluno: Roseval Donisete Malaquias Junior - RA: 758597
- Professor: Delano Medeiros Beder

## Introdução
O desenvolvimento de softwares orientados em larga escla apresenta dificuldades intrínsecas à alta complexidade desta atividade. Diante disso, Robert C. Martin (Uncle Bob) buscou elicitar um conjunto de postulados a serem seguidos na arquiteturação de softwares orientados a objetos a fim de facilitar o entendimento e desenvolvimento destes programas. Com isso, surgiu-se o termo SOLID como um acrônimo dos cinco postulados de design de programação, destinados ao desenvolvimento de softwares com maior flexibilidade de extensão e manutenção. Neste artigo, será feita a apresentação do **[S]** *Single Resposability Principle* (SRP) como o primeiro principio SOLID por meio da explicação conceitual do postulado e exemplificação de sua aplicação.

## Princípio da Resposabilidade Única
O SRP reza que todo módulo de código desenvolvido deve possuir apenas uma responsabilidade com relação as funcionalidade do programa, apresentando assim apenas um eixo de mudança. Os programas desenvolvidas com esse principio apresentam alta coesão e baixo acoplamento dado a especificidade dos blocos de código produzidos. Sendo assim, a manutenção destes programas será facilitados devido a baixa interdependência entre os diversos blocos de código e focalização das responsabilidade de cada bloco. Esses módulos podem ser melhor reutilizados com menores chances de encadeamento de erros, visto que o programa estará melhor organizado em módulos bem definidos.

Entretanto, a aplicação deste postulado pode ser complicada, sendo muito dependente da experiencia do programador para a detecção de responsabilidades no âmbito do programa em desenvolvimento. Como por exemplo, muitas vezes para duas pessoas diferentes, um mesmo bloco de código pode possuir um numero diferente de responsabilidades, sendo que as experiencias dessas duas pessoas na área podem ser diferentes, produzindo visões duas de uma mesma solução. Isso pode ser, visualizado na classe ``EmailSender`` a seguir.

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
...
```
