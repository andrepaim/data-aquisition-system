# Sistema Assíncrono de Aquisição de Dados

## Descrição do Projeto

O objetivo deste projeto é criar um servidor assíncrono que simula um sistema de aquisição de dados de sensores industriais. 

Cada conexão representa um sensor. O servidor deve ser capaz de lidar com múltiplas conexões (sensores) simultaneamente, receber dados de forma assíncrona e registrar esses dados em um arquivo único para cada sensor.

## Requisitos

1. **Servidor assíncrono**: Implemente um servidor TCP utilizando a biblioteca Boost.Asio para aceitar múltiplas conexões simultaneamente. Cada conexão representa um sensor industrial.

2. **Recepção de dados**: O servidor deve receber dados dos clientes (sensores) de forma assíncrona. Cada mensagem enviada pelos sensores deve conter informações como identificador do sensor, data/hora da leitura e a leitura do sensor.

3. **Registro de dados**: Todas as leituras dos sensores recebidas pelo servidor devem ser registradas em um arquivo único para cada sensor. Assegure-se de que o servidor possa lidar com o acesso concorrente aos arquivos de log.

4. **Cliente de simulação**: Implemente um cliente de simulação que possa se conectar ao servidor e enviar dados de forma assíncrona. O cliente deve ser capaz de simular diferentes sensores enviando leituras em intervalos aleatórios.

5. **Consulta de registros**: Implemente uma funcionalidade que permita a um cliente solicitar as últimas `n` leituras de um sensor específico, identificado pelo seu ID. O servidor deve responder com as `n` últimas leituras registradas para esse sensor.

## Formato das Mensagens

### Sensor para Servidor (Envio de Dados)

A mensagem deve ter o seguinte formato: `SENSOR_ID:DATA_HORA:LEITURA\r\n`. 

Por exemplo: `SENSOR_001:2023-05-11T15:30:00:78.5\r\n`.

### Cliente para Servidor (Solicitação de Registros)

A mensagem deve ter o seguinte formato: `GET:SENSOR_ID:NUMERO_DE_REGISTROS\r\n`. 
 
Por exemplo: `GET:SENSOR_001:10\r\n`.

### Servidor para Cliente (Resposta à Solicitação de Registros)

A mensagem deve ter o seguinte formato: `DATA_HORA:LEITURA\r\n...DATA_HORA:LEITURA\r\nEND\r\n`. 

O servidor deve retornar as `n` últimas leituras do sensor, precedidas pelo número total de registro, separadas por `;`. 

Por exemplo: `2;2023-05-11T15:30:00:78.5;2023-05-11T15:31:00:77.5\r\n`. 

Se um cliente solicitar mais registros do que os disponíveis, o servidor deve retornar apenas os registros disponíveis.

### Servidor para Cliente (Resposta de Erro) 

Se o ID do sensor fornecido pelo cliente for inválido (ou seja, não corresponder a nenhum sensor conhecido pelo servidor), o servidor deve enviar uma resposta de erro no seguinte formato: `ERROR:INVALID_SENSOR_ID\r\n`. 

Por exemplo: `ERROR:INVALID_SENSOR_ID\r\n`.

## Formato do Arquivo de Log

O arquivo de log é um arquivo binário composto por registros. Cada registro contém o ID do sensor (string), a data/hora da leitura (timestamp) e o valor da leitura (double). 

A definição do registro pode ser representada como uma struct em C++:

```c++
#pragma pack(push, 1)
struct LogRecord {
    char sensor_id[32]; // supondo um ID de sensor de até 32 caracteres
    std::time_t timestamp; // timestamp UNIX
    double value; // valor da leitura
};
#pragma pack(pop)
```

### Trabalhando com std::time_t

`std::time_t` é um tipo definido na biblioteca padrão de C++ que representa o tempo como o número de segundos passados desde a época Unix, que é 00:00:00 UTC em 1º de janeiro de 1970 (sem incluir os segundos bissextos). Portanto, é comumente usado para armazenar timestamps.

Para converter uma string de data/hora no formato "AAAA-MM-DDTHH:MM:SS" para `std::time_t`, você pode usar a seguinte função:

```c++
#include <ctime>
#include <iomanip>
#include <sstream>

std::time_t string_to_time_t(const std::string& time_string) {
    std::tm tm = {};
    std::istringstream ss(time_string);
    ss >> std::get_time(&tm, "%Y-%m-%dT%H:%M:%S");
    return std::mktime(&tm);
}
```

Para converter um std::time_t para uma string de data/hora no formato "AAAA-MM-DDTHH:MM:SS", você pode usar a seguinte função:

```c++
#include <ctime>
#include <iomanip>
#include <sstream>

std::string time_t_to_string(std::time_t time) {
    std::tm* tm = std::localtime(&time);
    std::ostringstream ss;
    ss << std::put_time(tm, "%Y-%m-%dT%H:%M:%S");
    return ss.str();
}```
