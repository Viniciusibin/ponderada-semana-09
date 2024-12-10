# ponderada-semana-09
# Relatório Técnico de Análise de Segurança em Solução IoT

## 1. Introdução

Este relatório apresenta a análise de segurança de uma solução IoT baseada no ESP32, seguindo o roteiro proposto. A análise foi realizada sobre o código fonte de um servidor web embarcado no ESP32, conforme o exemplo disponível em:  
[https://randomnerdtutorials.com/esp32-web-server-arduino-ide/](https://randomnerdtutorials.com/esp32-web-server-arduino-ide/)

O objetivo desta atividade é identificar vulnerabilidades, possíveis ataques, e realizar análises estática e dinâmica. Adicionalmente, foi elaborado um passo-a-passo para dois ataques exemplificativos, avaliando sua probabilidade, impacto e risco. Por fim, apresenta-se uma tabela consolidada com os ataques ordenados por criticidade, e a realização de um teste prático de ataque (análise dinâmica).

## 2. Metodologia

1. **Análise Estática do Código**: Examinamos o código fonte disponibilizado, avaliando o método de conexão Wi-Fi, a forma como os comandos HTTP são tratados, e a ausência de mecanismos de segurança (autenticação, criptografia, controle de acesso).
   
2. **Identificação de Vulnerabilidades**: Foram listados pontos fracos que podem ser explorados por um atacante, tais como exposição de credenciais, ausência de autenticação no controle de GPIOs, ausência de criptografia, e ausência de sanitização de dados.
   
3. **Definição de Possíveis Ataques**: Com base nas vulnerabilidades, identificamos ataques viáveis. Selecionamos pelo menos dois deles para detalhar o passo-a-passo, probabilidade, impacto e risco.
   
4. **Consolidação dos Resultados**: Organizamos as análises em um relatório técnico e criamos uma tabela consolidada, classificando os ataques do maior para o menor risco.
   
5. **Análise Dinâmica (Ir Além)**: Montagem do circuito em protoboard, compilação do código e realização de um ataque de forma manual para comprovar a viabilidade prática.

## 3. Análise Estática do Código

### Vulnerabilidades Identificadas

- **Exposição de Credenciais no Código-Fonte**: O SSID e a senha do Wi-Fi estão diretamente embutidos no código. Caso o firmware seja obtido por engenharia reversa, as credenciais da rede poderão ser facilmente extraídas.
  
- **Ausência de Autenticação para Controles Sensíveis**: Não há qualquer mecanismo de autenticação para alterar o estado dos GPIOs. Basta enviar uma requisição HTTP para ativar ou desativar um pino.
  
- **Tráfego não Criptografado (HTTP)**: A comunicação é feita em texto claro (HTTP), sem TLS/HTTPS. Um atacante na mesma rede pode interceptar e analisar tráfego, descobrindo parâmetros de controle.
  
- **Falta de Sanitização do Input**: A aplicação lê cabeçalhos HTTP diretamente em uma string (`header`) sem validação, criando potencial para injeção de dados maliciosos.
  
- **Susceptibilidade a Ataques de Negação de Serviço (DoS)**: Sem limitação de requisições, um atacante pode sobrecarregar o ESP32 com múltiplas conexões ou requisições simultâneas.

### Possíveis Ataques

- **Ataque A: Controle Não Autorizado dos GPIOs**  
  Explorando a ausência de autenticação, um atacante pode ligar ou desligar saídas do ESP32 apenas enviando requisições HTTP simples.
  
- **Ataque B: Exfiltração de Credenciais de Rede**  
  Obtendo o firmware ou código-fonte, um atacante descobre o SSID e senha, comprometendo a rede Wi-Fi e todos os dispositivos nela contidos.

Além destes, outros ataques possíveis incluem Negação de Serviço (DoS), Injeção de Código via Input HTTP, e Escalamento de Privilégios dentro da rede interna.

## 4. Descrição dos Ataques Selecionados

A seguir, detalhamos dois ataques exemplificativos:

### Ataque 1: Controle Não Autorizado dos GPIOs

**Passo-a-Passo:**
1. O atacante conecta-se à mesma rede Wi-Fi do ESP32 (caso tenha acesso ou as credenciais sejam conhecidas).
2. Descobre o IP do ESP32 (por exemplo, via scan de rede).
3. Envia uma requisição HTTP, por exemplo: `http://<IP_DO_ESP32>/26/on`
4. O pino GPIO 26 é ligado sem qualquer confirmação ou autenticação.

**Probabilidade:** Alta. Qualquer pessoa na mesma rede pode executar o ataque.  
**Impacto:** Médio-Alto. Pode desligar ou ligar dispositivos críticos, alterar estados de sensores ou atuadores, causando falhas operacionais.  
**Risco Resultante:** Alto, pois a facilidade de execução e o impacto potencial no sistema tornam a ameaça significativa.

### Ataque 2: Exfiltração de Credenciais de Rede

**Passo-a-Passo:**
1. O atacante obtém o código-fonte do dispositivo (por engenharia reversa do firmware ou acesso ao repositório do código).
2. Identifica o SSID e a senha do Wi-Fi no código.
3. Conecta-se à rede Wi-Fi comprometida.
4. A partir da rede, o atacante pode realizar ataques mais avançados (sniffing, escalonamento de privilégios, acesso a outros dispositivos internos).

**Probabilidade:** Média. Depende de acesso ao firmware ou código, mas não é difícil caso o atacante tenha acesso físico ao dispositivo ou ao repositório.  
**Impacto:** Alto. Comprometer a rede local expõe dados, sistemas e outros dispositivos sensíveis.  
**Risco Resultante:** Alto, pela gravidade do acesso indevido à rede inteira.

## 5. Tabela Consolidada dos Ataques (Ordem Decrescente de Risco)

Abaixo segue a tabela com alguns ataques identificados, ordenados pelo risco (do maior para o menor).


| Título do Ataque                        | Probabilidade | Impacto     | Risco |
|-----------------------------------------|---------------|-------------|-------|
| Exfiltração de Credenciais de Rede       | Média         | Alto        | Alto  |
| Controle Não Autorizado dos GPIOs        | Alta          | Médio-Alto  | Alto  |
| Negação de Serviço (DoS)                 | Alta          | Médio       | Alto  |
| Injeção de Código via Input HTTP         | Média         | Médio-Alto  | Alto  |
| Escalamento de Privilégios na Rede       | Média         | Alto        | Alto  |


Observação: Embora todos listados tenham alto risco, a ordem prioriza acesso à rede (impacto crítico), depois controle direto do dispositivo e, por fim, ataques com impacto operacional e possibilidade de expansão (privilégios).


| Ataque                                     | Descrição                                                                                                     | Passo-a-Passo do Ataque                                                                                                                                                          | Probabilidade | Impacto        | Risco | Justificativa do Risco                                                                                           | Recomendações                                                                                           |
|--------------------------------------------|---------------------------------------------------------------------------------------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|---------------|----------------|-------|---------------------------------------------------------------------------------------------------------------|---------------------------------------------------------------------------------------------------------|
| **Controle Não Autorizado dos GPIOs**      | Uso de requisições HTTP sem autenticação para alterar estados dos pinos GPIO (ex: ligar/desligar dispositivos)| 1. Atacante conecta-se à mesma rede Wi-Fi.<br>2. Envia requisições HTTP para rotas como `/26/on` ou `/27/off`.<br>3. Obtém controle funcional do equipamento.                       | Alta          | Médio-Alto     | Alto  | A ausência de autenticação expõe diretamente a funcionalidade do dispositivo, permitindo que qualquer um o controle livremente. | Implementar autenticação (por exemplo, via token ou credenciais).<br>Introduzir autorização por IP ou MAC. |
| **Exfiltração de Credenciais de Rede**     | Obtenção de SSID e senha do Wi-Fi embutidas no código, possibilitando acesso indevido à rede local            | 1. Atacante obtém acesso ao código-fonte ou ao binário do firmware.<br>2. Extrai SSID/senha do Wi-Fi.<br>3. Conecta-se à rede, comprometendo outros dispositivos.                 | Média         | Alto           | Alto  | O acesso à rede pode comprometer múltiplos dispositivos, expondo dados sensíveis e sistemas críticos.              | Armazenar credenciais de forma segura (criptografadas).<br>Implementar provisioning seguro (via OTA). |
| **Negação de Serviço (DoS)**                | Envio de requisições em massa ou manutenção de conexões pendentes para sobrecarregar o servidor do ESP32       | 1. Enviar múltiplas requisições simultâneas.<br>2. Manter conexões abertas além do tempo limite.<br>3. Forçar o dispositivo a esgotar recursos (memória, CPU).                      | Alta          | Médio          | Alto  | O ESP32 possui recursos limitados, sendo facilmente saturado por tráfego excessivo, indisponibilizando o serviço. | Limitar número de conexões simultâneas.<br>Implementar timeouts mais restritivos e fila de requisições. |
| **Injeção de Código via Input HTTP**        | Inserção de valores maliciosos nos cabeçalhos HTTP, aproveitando a falta de validação e sanitização           | 1. Enviar requisição com cabeçalhos manipulados.<br>2. Explorar a concatenação direta do input a variáveis.<br>3. Induzir comportamento não previsto (embora limitado).             | Média         | Médio-Alto     | Alto  | Sem sanitização adequada, a aplicação pode processar dados inesperados, potencialmente afetando a lógica de controle. | Validar e sanitizar toda entrada HTTP.<br>Adotar bibliotecas seguras de parsing de requests.           |
| **Escalamento de Privilégios na Rede**      | Uso do ESP32 comprometido como ponto de partida para atacar outros dispositivos na LAN                         | 1. Comprometer o ESP32 (via credenciais ou GPIOs).<br>2. Mapeamento da rede local e procura por vulnerabilidades em outros hosts.<br>3. Acesso e controle de sistemas internos.   | Média         | Alto           | Alto  | Após ter o controle do ESP32, um atacante pode explorar a rede interna, ampliando o escopo do ataque.              | Isolar o dispositivo em rede segregada (VLAN).<br>Aplicar firewall e monitoramento do tráfego interno. |


| Ataque                                     | Descrição                                                                                                     | Passo-a-Passo do Ataque                                                                                                                                                          | Probabilidade | Impacto        | Risco | Justificativa do Risco                                                                                           | Recomendações                                                                                           |
|--------------------------------------------|---------------------------------------------------------------------------------------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|---------------|----------------|-------|---------------------------------------------------------------------------------------------------------------|---------------------------------------------------------------------------------------------------------|
| *Controle Não Autorizado dos GPIOs*      | Uso de requisições HTTP sem autenticação para alterar estados dos pinos GPIO (ex: ligar/desligar dispositivos)| 1. Atacante conecta-se à mesma rede Wi-Fi.<br>2. Envia requisições HTTP para rotas como /26/on ou /27/off.<br>3. Obtém controle funcional do equipamento.                       | Alta          | Médio-Alto     | Alto  | A ausência de autenticação expõe diretamente a funcionalidade do dispositivo, permitindo que qualquer um o controle livremente. | Implementar autenticação (por exemplo, via token ou credenciais).<br>Introduzir autorização por IP ou MAC. |
| *Exfiltração de Credenciais de Rede*     | Obtenção de SSID e senha do Wi-Fi embutidas no código, possibilitando acesso indevido à rede local            | 1. Atacante obtém acesso ao código-fonte ou ao binário do firmware.<br>2. Extrai SSID/senha do Wi-Fi.<br>3. Conecta-se à rede, comprometendo outros dispositivos.                 | Média         | Alto           | Alto  | O acesso à rede pode comprometer múltiplos dispositivos, expondo dados sensíveis e sistemas críticos.              | Armazenar credenciais de forma segura (criptografadas).<br>Implementar provisioning seguro (via OTA). |
| *Negação de Serviço (DoS)*                | Envio de requisições em massa ou manutenção de conexões pendentes para sobrecarregar o servidor do ESP32       | 1. Enviar múltiplas requisições simultâneas.<br>2. Manter conexões abertas além do tempo limite.<br>3. Forçar o dispositivo a esgotar recursos (memória, CPU).                      | Alta          | Médio          | Alto  | O ESP32 possui recursos limitados, sendo facilmente saturado por tráfego excessivo, indisponibilizando o serviço. | Limitar número de conexões simultâneas.<br>Implementar timeouts mais restritivos e fila de requisições. |
| *Injeção de Código via Input HTTP*        | Inserção de valores maliciosos nos cabeçalhos HTTP, aproveitando a falta de validação e sanitização           | 1. Enviar requisição com cabeçalhos manipulados.<br>2. Explorar a concatenação direta do input a variáveis.<br>3. Induzir comportamento não previsto (embora limitado).             | Média         | Médio-Alto     | Alto  | Sem sanitização adequada, a aplicação pode processar dados inesperados, potencialmente afetando a lógica de controle. | Validar e sanitizar toda entrada HTTP.<br>Adotar bibliotecas seguras de parsing de requests.           |
| *Escalamento de Privilégios na Rede*      | Uso do ESP32 comprometido como ponto de partida para atacar outros dispositivos na LAN                         | 1. Comprometer o ESP32 (via credenciais ou GPIOs).<br>2. Mapeamento da rede local e procura por vulnerabilidades em outros hosts.<br>3. Acesso e controle de sistemas internos.   | Média         | Alto           | Alto  | Após ter o controle do ESP32, um atacante pode explorar a rede interna, ampliando o escopo do ataque.              | Isolar o dispositivo em rede segregada (VLAN).<br>Aplicar firewall e monitoramento do tráfego interno. |
