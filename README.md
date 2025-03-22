# Script Performance da ViaCep
#### Desafio de um processo seletivo para Analista de Performance de Software
### O que precisa para editar e executar?
- Acesso à internet
- Acesso a um hardware que suporte a execução do teste
- JMeter
- JDK
- Editor de texto ou planilha (ex: Bloco de Notas, Notepad++, Visual Studio Code, Excel)


### O que é?
- Um teste de carga em .jmx, com uma massa de dados em CSV
- O script acessa 2 endpoints da API da ViaCep
- O script foi originalmente desenvolvido para um processo seletivo e cumpre comm as especifidades do desafio da empresa em questão


### Edite o Script
<b>1- HTTP Request Default: É onde coloco valores que estarão em todos os endpoints, como:</b>
- protocolo (protocol[HTPP], que coloquei o "HTTPS") 
- DNS (Server Name or IP, que coloquei o "viacep.com.br")
#### Opcionalmente, pode-se colocar:
- A porta que deseja acessar, geralmente são 443, 80 ou 8080
- o Path, que também aceita query strings, caso opte por não editar como parâmetro ou "body data"
- Content encoding, que geralmente é UTF-8
<br>

2- CSV Data Set Config: É onde configuramos a massa de dados, com:
- caminho do arquivo de massa de dados em "Filename"
- tipo de encoding em "File encoding", geralmente é UTF-8
- nome das variáveis, na ordem que os dados são colocados no CSV em "Variable Names"
- como estão separados os elementos em "Delimeter"

3- Thread Group: É onde se configura a carga, ramp-up e tempo de teste

4- Uniform Random Timer: É onde se configura o Think Timer, colocando os valores mínimos e máximos de tempo

5- Throughput Controller: São as controllers que permitem configurar quanto porcento das execuções de threads os http requests filhos irão executar

6- HTTP Request: É onde se configura o acesso a cada endpoint desejado

7- Response Assertion: É onde se configura falsos positivo (erro tratado no response) ou falso negativo (validar que retornou o erro esperado no projeto)

8- View Results Tree: Respostas das requisições, mostrando o que foi enviado e recebido, tanto no body quanto no header

9- Summary Report: Dados todas de cada tipo de requisição, mostrando os valores totais de samples, média, mínima, máxima, porcentagem de erro, etc

## Configurações deste script
### HTTP Resquest Defaults:
Considerando que o site é sempre https://viacep.com.br
- Protocolo: https
- Server: viacep.com.br
<br>

### User Defined Variables:
- Valores mockados para smoke tests (esta desabilitado, para que o script use apenas a massa CSV)
<br>

### CSV Data Set Config:
- Coloque o caminho do CSV em "Filename",
- Se mudar o nome das variáveis, deve-se mudar onde são sendo chamadas, nos paths das HTTP Request
- Se mudar o "Recycle on EOF", o script irá parar assim que chegar a última linha da tabela, mas como a massa "não queima", mantemos ele em "true" para que o script continue rodando e acesse novamente os valores do CSV em questão
<br>

### Thread Group:
- Configurado Ramp-up de 30minutos + 1hora de Steady State
- Com threads simultâneas para acessar pelo menos 1000 requisições ao longo de 1hora em Steady State
<br>

### Uniform Random Timer:
- Random Delay Maximum,sendo o valor máximo: 1000ms
- Constant Delay Offset, sendo o valor mínimo: 100ms
<br>

### Throughput Controllers:
- Configurei ambas com "Percent Execution"
- A primeira: 60
- A segunda: 40
- Desta forma, será distribuído as requisições em 60% para o primeiro endpoint e 40% para o segundo
<br>

### HTTP Request: 
Neste teste, acessamos 2, sendo eles:
- GET /ws/${cep}/json/
- GET /ws/${state}/${city}/${address}/json/

### Response Assertions:
Para o primeiro endpoint, o retorno de quando o CEP não está no formato correto, é o HTTP 400, então não exige nenhuma configuração adicional, entretanto se o CEP for inválido (conforme a base de dados da Viacep) mas no formato correto, a resposta é o seguinte json:
<br>
{
   "erro":"true"
}

Por ser um erro tratado, a asserção deste endpoint está configurada para o JMeter mostrar como erro ao identificar a palavra "erro", mesmo retornando http 200.

Para o segundo endpoint, consideramos o que a aplicação não reconehce como lugar válido ao passar o UF, cidade e logradouro (váriaveis declaradas como: state, city e address, respectivamente), e ao ter um erro, retorna: []
Então, a asserção deste endpoint está configurada para o JMeter mostrar como erro ao identificar os caracteres "[]", mesmo retornando http 200.
