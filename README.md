# Trabalho 01 - Princípio da Responsabilidade Única
- Universidade Federal de São Carlos - Departamento de Computação
- Programação Orientada a Objetos Avançada
- Aluno: Roseval Donisete Malaquias Junior - RA: 758597
- Professor: Delano Medeiros Beder

## Introdução
O desenvolvimento de softwares orientados em larga escla apresenta dificuldades intrínsecas à alta complexidade desta atividade. Diante disso, Robert C. Martin (Uncle Bob) buscou elicitar um conjunto de postulados a serem seguidos na arquiteturação de softwares orientados a objetos a fim de facilitar o entendimento e desenvolvimento destes programas. Com isso, surgiu-se o termo SOLID como um acrônimo dos cinco postulados de design de programação, destinados ao desenvolvimento de softwares com maior flexibilidade de extensão e manutenção. Neste artigo, será feita a apresentação do **[S]** *Single Resposability Principle* (SRP) como o primeiro principio SOLID por meio da explicação conceitual do postulado e exemplificação de sua aplicação.

## Princípio da Resposabilidade Única
O SRP reza que todo módulo de código desenvolvido deve possuir apenas uma responsabilidade com relação as funcionalidade do programa, apresentando assim apenas um eixo de mudança. Os programas desenvolvidas com esse principio apresentam alta coesão e baixo acoplamento dado a especificidade dos blocos de código produzidos. Sendo assim, a manutenção destes programas será facilitados devido a baixa interdependência entre os diversos blocos de código e focalização das responsabilidade de cada bloco. Esses módulos podem ser melhor reutilizados com menores chances de encadeamento de erros, visto que o programa estará melhor organizado em módulos bem definidos.

Entretanto, a aplicação deste postulado pode ser complicada, sendo muito dependente da experiencia do programador para a detecção de responsabilidades no âmbito do programa em desenvolvimento. Como por exemplo, muitas vezes para duas pessoas diferentes, um mesmo bloco de código pode possuir um numero diferente de responsabilidades, sendo que as experiencias dessas duas pessoas na área podem ser diferentes, produzindo visões diferentes de uma mesma solução. Diante disso, dentro do desenvolvimento de software é possível ocorrer a má utilização do SRP, como por exemplo nesta classe ''EmailSender'', que funciona para enviar emails a clientes em um sistema web de compras online.
