# Plugin Stack Recommender

##  Menu

- [Requisitos](#requisitos)
- [Executando o WebService](#executando-o-webservice)
- [Executando o Plugin](#executando-o-plugin)
- [Links Importantes](#links-importantes)

## Requisitos
- Java 11
- [Eclipse IDE for Enterprise Java and Web Developers](https://www.eclipse.org/downloads/packages/release/2021-09/r/eclipse-ide-enterprise-java-and-web-developers)
- Tomcat 8.5
- MySQL 5.7.36

## Executando o WebService

### Importando o projeto no Eclipse
Importe o projeto ***stacktracewebservice*** no eclipse:

***import > general > projects from folder or archive***

Selecione o diretório onde está o projeto e clique em finish

O projeto necessita que você importe também os projetos ***stacktracemodelplugin*** e o projeto ***stacklibs***. Execute os mesmos passos citados anteriormente para a importação.

Após a importação de todas as bibliotecas, você precisará adicionar o Tomcat ao build path da aplicação:

***stacktracewebservice > botão direito > build path > configure build path***

Selecione a lib ***server runtime***, escolha ***edit*** no painel do lado direito e escolha o tomcat que aparece na lista.

### Corrigindo dependências

Como o projeto foi feito em Java 7, algumas funções foram removidas da linguagem no Java 9, então é necessário adicionar algumas dependências no arquivo ***pom.xml*** para recuperar essas funções sem precisar reescrever todo o código. Adicione a seguinte dependência ao arquivo:
```xml
<!-- API, java.xml.bind module -->
<dependency>
	<groupId>jakarta.xml.bind</groupId>
	<artifactId>jakarta.xml.bind-api</artifactId>
	<version>2.3.2</version>
</dependency>
<!-- Runtime, com.sun.xml.bind module -->
<dependency>
	<groupId>org.glassfish.jaxb</groupId>
	<artifactId>jaxb-runtime</artifactId>
	<version>2.3.2</version>
</dependency>
```

### Configurando banco de dados

Foi utilizado o banco de dados ***MySQL 5.7.36*** para a execução da aplicação, e é recomendado que utilize-se o mesmo para evitar possíveis problemas na execução. É necessário a criação de um banco de dados usando alguma ferramenta gráfica como o ***DBeaver***, ***MySQLWorkbench*** ou utilizando o próprio console do ***MySql***. Tendo criado o banco, será necessário inserir as informações relativas à conexão no arquivo. Modifique o arquivo com as informações:

```xml
<property name="hibernate.connection.url">jdbc:mysql://localhost:3306/${nome_do_seu_banco}</property>	
<property name="hibernate.connection.username">${usuario_do_seu_banco}</property>
<property name="hibernate.connection.password">${senha_do_seu_banco}</property>
```
Modifique a linha da propriedade ***hibernate.show_sql*** para ***true***, para facilitar a visualização que o programa está executando corretamente:

```xml
<property name="hibernate.show_sql">false</property>
```

Adicione a propriedade ***hibernate.hbm2ddl.auto*** para que o hibernate possa gerar as tabelas automaticamente:

```xml
<property name="hibernate.hbm2ddl.auto">update</property>
```

### Corrigindo erros

Algumas informações das tabelas não estão sendo inseridas corretamente. Para corrigir isto, é necessária uma alteração no arquivo ***StackoverflowAPIManager***. Porém, este arquivo foi empacotado como um ***jar*** no projeto ***stacklibs***, e não possui o código fonte, logo, foi necessário descompilar a classe para realizar as alterações.

A url de consulta existente, não estava retornando todos os campos necessários para a execução correta do programa, então foi necessário adicionar o seguinte trecho de código para corrigir esse problema:

```java
static {
	StackoverflowAPIManager.keyapp = "qR5IDEL3*mrVv6DT*F2Pog((";
	StackoverflowAPIManager.baseurl = "https://api.stackexchange.com/2.2/search?tagged=hibernate&site=stackoverflow&filter=!Pw)h(Uw4R_.Nt-fTg3-lrhgDr978Xe&key=" + 	StackoverflowAPIManager.keyapp;
	StackoverflowAPIManager.baseurl_fromdate = "https://api.stackexchange.com/2.2/search?fromdate=data_inicio&tagged=hibernate&site=stackoverflow&filter=!Pw)h(Uw4R_.Nt-fTg3-lrhgDr978Xe&key=" + StackoverflowAPIManager.keyapp;
	StackoverflowAPIManager.pagesize = 100;
}
```

Uma cópia da classe modificada está na pasta raiz do repositório. Adicione a classe ao pacote ***br.ufrn.lets.stacktracewebservice.util*** e atualize os imports que utilizam a classe, para apontar para esta nova classe.

### Executando o projeto
Vá ao arquivo ***ServletListener*** e descomente a linha que chama o método que realiza as consultas à API:

```java
StackoverflowTimerExecute.getInstance(5000L).start();//2628000000L
```
Execute o webservice:

***stacktracewebservice > botão direito > run as > run on server***

Escolha o Tomcat 8.5 na tela que aparece, clique em next e escolha a pasta onde está o Tomcat no seu computador. Um server será criado no Eclipse.

Feito isto, o programa deverá começar a fazer as requisições para a API do stackoverflow. Esta etapa pode demorar.

### Criando tabela de recomendações
Para o funcionamento correto do plugin, é necessária a criação de uma tabela de recomendações. Na pasta ***scripts*** do projeto, existe um arquivo com extensão ***.sql*** que pode ser utilizado para a criação desta tabela. É necessário algumas modificações para que o script execute corretamente. Uma cópia do arquivo modificado está na pasta raiz do repositório.

### Bugs existentes
O programa começa a pegar as perguntas do dia 14/09/2014 (esta data pode ser alterada), porém, ao chegar no dia atual, o programa começa a verificar as perguntas novamente a partir da data anterior, sempre inserindo no banco stacktraces repetidas.

Caso você pare a execução do programa, não conseguirá fazê-lo executar de novo corretamente sem antes apagar os dados da tabela ***stackoverflow_timer***.

## Executando o Plugin

### Importando o projeto no Eclipse
Importe o projeto ***stackrecommender*** no eclipse:

***import > general > projects from folder or archive***

Selecione o diretório onde está o projeto e clique em finish

O projeto necessita que você importe também o projeto ***stacktracemodelplugin***. Execute os mesmos passos citados anteriormente para a importação.

### Executando o projeto
Execute o projeto:

***stackrecommender > botão direito > run as > eclipse application***

Uma nova janela do Eclipse irá abrir. Essa janela estará executando o Eclipse com o plugin. Crie um novo projeto Java simples para testar as funcionalidades do plugin.

### Configurações do plugin
Ao executar o plugin, ele dará um erro no terminal, pois a porta que ele espera conectar com o webservice, é a porta 9090, e a porta padrão do Tomcat que executa o webservice, é a porta 8080. Podemos modificar a porta utilizada pelo plugin nas configurações:

***window > preferences > stackrecommender***

Modifique a porta para 8080, e aplique as alterações. O botão de testar a conexão aparenta não estar funcionando corretamente, mas a configuração deve funcionar.
Ao aplicar as alterações, pode-se usar ***ctrl + espaço*** para obter sugestões a partir dos métodos digitados no código. Também é possível realizar buscas manuais por exceções e visualizar as perguntas dentro da própria IDE. Para isso, habilite as views:

***window > show view > other > stackrecommender***

Escolha as views desejadas e elas aparecerão no canto inferior da IDE.

## Links Importantes

- [Documentação original da ferramenta feita por Teresa](https://docs.google.com/document/d/15Iv3z1-pHAtvSX8h6tW-sRz1JH2MF5Q9HrzUBScQYUY/edit)
- [Apresentação da ferramenta na defesa do mestrado de Teresa](https://docs.google.com/presentation/d/1IwskfisDV4HKkDchb4gv4ienSBG8uGm9GJ9KVOnDSeA/edit#slide=id.p)
- [Apresentação do plugin de recomendações feito por Teresa](https://docs.google.com/presentation/d/1xymcVAPneaPwplpMP-8pZLxiUW3wFMj06Kgm4C00O7w/edit#slide=id.p)
- [Dissertação de Teresa](https://github.com/paulosandinof/stackrecommender/files/7406718/Dissertacao_versao_final.pdf)
