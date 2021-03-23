## Working with Apache Thrift

# 1. Visão Geral
Neste artigo, descobriremos como desenvolver aplicativos cliente-servidor de plataforma cruzada com a ajuda da estrutura RPC chamada Apache Thrift.

Vamos cobrir:

Definindo tipos de dados e interfaces de serviço com IDL
Instalando a biblioteca e gerando as fontes para diferentes idiomas
Implementando as interfaces definidas em um idioma específico
Implementando software cliente / servidor
Se você quiser ir direto para os exemplos, vá direto para a seção 5.

# 2. Apache Thrift
O Apache Thrift foi originalmente desenvolvido pela equipe de desenvolvimento do Facebook e atualmente é mantido pela Apache.

Em comparação com os Buffers de protocolo, que gerenciam os processos de serialização / desserialização de objetos entre plataformas, o Thrift se concentra principalmente na camada de comunicação entre os componentes do seu sistema.

Thrift usa uma linguagem de descrição de interface (IDL) especial para definir tipos de dados e interfaces de serviço que são armazenados como arquivos .thrift e usados ​​posteriormente como entrada pelo compilador para gerar o código-fonte de software cliente e servidor que se comunicam em diferentes linguagens de programação.

Para usar o Apache Thrift em seu projeto, adicione esta dependência Maven:

```
<dependency>
    <groupId>org.apache.thrift</groupId>
    <artifactId>libthrift</artifactId>
    <version>0.10.0</version>
</dependency>
```

Você pode encontrar a versão mais recente no repositório Maven.

# 3. Linguagem de descrição da interface
Conforme já descrito, o IDL permite definir as interfaces de comunicação em uma linguagem neutra. Abaixo você encontrará os tipos suportados atualmente.

### 3.1. Tipos de Base
- bool - um valor booleano (verdadeiro ou falso);
- byte - um inteiro assinado de 8 bits;
- i16 - um inteiro assinado de 16 bits;
- i32 - um inteiro assinado de 32 bits;
- i64 - um inteiro assinado de 64 bits;
- double - um número de ponto flutuante de 64 bits;
- string - uma string de texto codificada usando a codificação UTF-8.

### 3.2. Tipos Especiais
- binário - uma sequência de bytes não codificados;
- opcional - um tipo opcional de Java 8.

### 3.3. Structs
Freestar
Thrift structs são equivalentes a classes em linguagens OOP, mas sem herança. Uma estrutura tem um conjunto de campos fortemente tipados, cada um com um nome exclusivo como identificador. Os campos podem ter várias anotações (IDs de campo numérico, valores padrão opcionais, etc.).

### 3.4. Recipientes
Contêineres Thrift são contêineres fortemente tipados:

- list - uma lista ordenada de elementos;
- conjunto - um conjunto não ordenado de elementos únicos
- map <type1, type2> - um mapa de chaves estritamente exclusivas para valores
Os elementos do contêiner podem ser de qualquer tipo válido de Thrift.

### 3.5. Exceções
As exceções são funcionalmente equivalentes a structs, exceto que herdam das exceções nativas.

### 3.6. Serviços
Os serviços são, na verdade, interfaces de comunicação definidas usando tipos Thrift. Eles consistem em um conjunto de funções nomeadas, cada uma com uma lista de parâmetros e um tipo de retorno.

# 4. Geração de código-fonte
### 4.1. Suporte de linguas
Há uma longa lista de idiomas suportados atualmente:

- C ++;
- C #;
- Vai;
- Haskell;
- Java;
- Javascript;
- Node.js;
- Perl;
- PHP;
- Pitão;
- Ruby.
Você pode verificar a lista completa aqui.

### 4.2. Usando o arquivo executável da biblioteca
Basta baixar a versão mais recente, compilá-la e instalá-la se necessário e usar a seguinte sintaxe:

```
cd path/to/thrift
thrift -r --gen [LANGUAGE] [FILENAME]
```

Nos comandos definidos acima, [LANGUAGE] é um dos idiomas suportados e [FILENAME] é um arquivo com definição IDL.

Observe o sinalizador -r. Diz ao Thrift para gerar o código recursivamente assim que perceber a inclusão em um determinado arquivo .thrift.

### 4.3. Usando o plugin Maven
Adicione o plugin em seu arquivo pom.xml:

```
<plugin>
   <groupId>org.apache.thrift.tools</groupId>
   <artifactId>maven-thrift-plugin</artifactId>
   <version>0.1.11</version>
   <configuration>
      <thriftExecutable>path/to/thrift</thriftExecutable>
   </configuration>
   <executions>
      <execution>
         <id>thrift-sources</id>
         <phase>generate-sources</phase>
         <goals>
            <goal>compile</goal>
         </goals>
      </execution>
   </executions>
</plugin>
```

Depois disso, basta executar o seguinte comando:

```
mvn clean install
```

Observe que este plugin não terá mais nenhuma manutenção. Visite esta página para obter mais informações.

# 5. Exemplo de um aplicativo cliente-servidor
### 5.1. Definindo Arquivo Thrift
Vamos escrever um serviço simples com exceções e estruturas:

```
namespace cpp com.isac.thrift.impl
namespace java com.isac.thrift.impl

exception InvalidOperationException {
    1: i32 code,
    2: string description
}

struct CrossPlatformResource {
    1: i32 id,
    2: string name,
    3: optional string salutation
}

service CrossPlatformService {

    CrossPlatformResource get(1:i32 id) throws (1:InvalidOperationException e),

    void save(1:CrossPlatformResource resource) throws (1:InvalidOperationException e),

    list <CrossPlatformResource> getList() throws (1:InvalidOperationException e),

    bool ping() throws (1:InvalidOperationException e)
}
```

Como você pode ver, a sintaxe é bastante simples e autoexplicativa. Definimos um conjunto de namespaces (por linguagem de implementação), um tipo de exceção, uma estrutura e, finalmente, uma interface de serviço que será compartilhada entre diferentes componentes.

Em seguida, basta armazená-lo como um arquivo service.thrift.

### 5.2. Compilando e gerando um código
Agora é hora de executar um compilador que irá gerar o código para nós:

```
thrift -r -out generated --gen java /path/to/service.thrift
```

Como você pode ver, adicionamos um sinalizador especial -out para especificar o diretório de saída para os arquivos gerados. Se você não obteve nenhum erro, o diretório gerado conterá 3 arquivos:

- CrossPlatformResource.java;
- CrossPlatformService.java;
- InvalidOperationException.java.

Vamos gerar uma versão C ++ do serviço executando:

```
thrift -r -out generated --gen cpp /path/to/service.thrift
```

Agora temos 2 implementações válidas diferentes (Java e C ++) da mesma interface de serviço.

### 5.3. Adicionando uma Implementação de Serviço
Embora Thrift tenha feito a maior parte do trabalho para nós, ainda precisamos escrever nossas próprias implementações do CrossPlatformService. Para fazer isso, precisamos apenas implementar uma interface CrossPlatformService.Iface:

```
public class CrossPlatformServiceImpl implements CrossPlatformService.Iface {

    @Override
    public CrossPlatformResource get(int id) 
      throws InvalidOperationException, TException {
        return new CrossPlatformResource();
    }

    @Override
    public void save(CrossPlatformResource resource) 
      throws InvalidOperationException, TException {
        saveResource();
    }

    @Override
    public List<CrossPlatformResource> getList() 
      throws InvalidOperationException, TException {
        return Collections.emptyList();
    }

    @Override
    public boolean ping() throws InvalidOperationException, TException {
        return true;
    }
}
```

### 5.4. Escrevendo um servidor
Como dissemos, queremos construir um aplicativo cliente-servidor de plataforma cruzada, portanto, precisamos de um servidor para isso. A melhor coisa sobre o Apache Thrift é que ele tem sua própria estrutura de comunicação cliente-servidor, o que torna a comunicação um pedaço de bolo:

```
public class CrossPlatformServiceServer {
    public void start() throws TTransportException {
        TServerTransport serverTransport = new TServerSocket(9090);
        server = new TSimpleServer(new TServer.Args(serverTransport)
          .processor(new CrossPlatformService.Processor<>(new CrossPlatformServiceImpl())));

        System.out.print("Starting the server... ");

        server.serve();

        System.out.println("done.");
    }

    public void stop() {
        if (server != null && server.isServing()) {
            System.out.print("Stopping the server... ");

            server.stop();

            System.out.println("done.");
        }
    }
}
```

A primeira coisa é definir uma camada de transporte com a implementação da interface TServerTransport (ou classe abstrata, para ser mais preciso). Já que estamos falando sobre servidor, precisamos fornecer uma porta para escutar. Em seguida, precisamos definir uma instância de TServer e escolher uma das implementações disponíveis:

- TSimpleServer - para servidor simples;
- TThreadPoolServer - para servidor multithread;
- TNonblockingServer - para servidor multithread sem bloqueio.

E, finalmente, forneça uma implementação de processador para o servidor escolhido que já foi gerado para nós pelo Thrift, ou seja, a classe CrossPlatofformService.Processor.

### 5.5. Escrevendo um cliente
E aqui está a implementação do cliente:

```
TTransport transport = new TSocket("localhost", 9090);
transport.open();

TProtocol protocol = new TBinaryProtocol(transport);
CrossPlatformService.Client client = new CrossPlatformService.Client(protocol);

boolean result = client.ping();

transport.close();
```

Da perspectiva do cliente, as ações são muito semelhantes.

Em primeiro lugar, defina o transporte e aponte-o para a nossa instância do servidor, depois escolha o protocolo adequado. A única diferença é que aqui inicializamos a instância do cliente que foi, mais uma vez, já gerada pelo Thrift, ou seja, a classe CrossPlatformService.Client.

Uma vez que é baseado em definições de arquivo .thrift, podemos chamar diretamente os métodos descritos lá. Neste exemplo específico, client.ping() fará uma chamada remota para o servidor que responderá com true.

# 6. Conclusão
Neste artigo, mostramos os conceitos básicos e as etapas para trabalhar com o Apache Thrift e mostramos como criar um exemplo de trabalho que utiliza a biblioteca Thrift.