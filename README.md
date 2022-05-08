# Apresentar o estado das linhas do Metropolitano de Lisboa no Home-Assistant
---

A integração do Home-Assistant para apresentar o estado das linhas do Metropolitano  de Lisboa obtém os dados na API do Metropolitano de Lisboa.

Trata-se de uma possivel solução. Usa o secrets.yaml, o configuration.yaml, algum templating com [Jinja](https://palletsprojects.com/p/jinja/), conseguidos a custo com uma pitada de cURL em Zsh.

Para isto tudo funcionar, precisam (além de ter o vosso Home-assistant instalado, dhãn!):
1. Criar uma conta de acesso à API do Metropolitano de Lisboa;
2. Criar uma Application e chaves de produção na API do Metropolitano de Lisboa;
3. Inserir o segredo obtido na API do Metropolitano de Lisboa no secrets.yaml;
4. Criar os sensores no configuration.yaml

Instruções específicas em cada um dos pontos seguintes.

## Como funciona tudo

1. Para aceder à API do Metropolitano de Lisboa necessitam de gerar um token temporário de N em N segundos que usa o token não temporário para autenticação, a que chamámos metro_secret.
2. A cada N segundos que tenham configurdo para o token temporario, o token temporario de acesso aos serviços muda.
3. Para obter o token temporário novo, há um sensor que a unica função é usar o metro_secret e guardar o token temporario de acesso.

## 1. Criar uma conta de acesso à API do Metropolitano de Lisboa

1. Registar utilizador no site da [API do Metropolitano de Lisboa](https://api.metrolisboa.pt/store/);

## 2. Criar uma Application e chaves de produção na API do Metropolitano de Lisboa

1. Entrar com a nova conta no site https://api.metrolisboa.pt/store/site/pages/login.jag
2. Criar aplicação em https://api.metrolisboa.pt/store/site/pages/applications.jag
3. Editar aplicação 
4. Gerar Consumer Key e Consumer Secret 
5. Mostrar chabes clicando no botão "Show keys"
6. Copiar o Basic token do segundo exemplo que se encontra junto ao -H "Authorization:

        curl -k -d "grant_type=client_credentials" \
        -H "Authorization: Basic IstoÉumSegredoMuitoGrandeCheioDeCaracteres" \
         https://api.metrolisboa.pt:8243/token
         
## 3. Inserir o segredo obtido na API do Metropolitano de Lisboa no secrets.yaml;

1. Abrir secrets.yaml;
2. inserir uma linha com a entrada "metro_secret":

        metro_secret: 'Basic IstoÉumSegredoMuitoGrandeCheioDeCaracteres'

3. Gravar e fechar.

## 4. Criar os sensores no configuration.yaml

1. Abrir configuration.yaml;
2. Copiar código:

        - platform: rest
          resource: https://api.metrolisboa.pt:8243/token
          value_template: '{{ value_json.access_token }}'
          method: POST
          payload: "grant_type=client_credentials"
          name: "Metro Bearer token"
          headers:
              Content-Type: application/x-www-form-urlencoded
              Authorization: !secret metro_secret
          scan_interval: 3600

        - platform: rest
          resource: https://api.metrolisboa.pt:8243/estadoServicoML/1.0.1/estadoLinha/todos
          #value_template: '{{ value_json.resposta }}'
          name: "Linha do Metro"
          json_attributes:
            - resposta
            - codigo
          headers: 
              accept: 'application/json'
              Authorization: 'Bearer {{ states.sensor.metro_bearer_token.state }}'
              
        - platform: template
          sensors:
              codigo:
                  friendly_name: "Código de resposta do metro"
                  value_template: "{{ state_attr('sensor.linha_do_metro', 'codigo') }}"
              estado_azul:
                  friendly_name: "Estado da linha azul metro"
                  value_template: "{{ state_attr('sensor.linha_do_metro', 'resposta').azul }}"
              tipo_msg_az:
                  friendly_name: "Tipo de mensagem azul metro"
                  value_template: "{{ state_attr('sensor.linha_do_metro', 'resposta').tipo_msg_az }}"
              estado_amarela:
                  friendly_name: "Estado da linha amarela metro"
                  value_template: "{{ state_attr('sensor.linha_do_metro' , 'resposta').amarela }}"
              tipo_msg_am:
                  friendly_name: "Tipo de mensagem amarelo metro"
                  value_template: "{{ state_attr('sensor.linha_do_metro', 'resposta').tipo_msg_am }}"
              estado_vermelha:
                  friendly_name: "Estado da linha vermelha metro"
                  value_template: "{{ state_attr('sensor.linha_do_metro', 'resposta').vermelha }}"
              tipo_msg_vm:
                  friendly_name: "Tipo de mensagem vermelha metro"
                  value_template: "{{ state_attr('sensor.linha_do_metro', 'resposta').tipo_msg_vm }}"
              estado_verde:
                  friendly_name: "Estado da linha verde metro"
                  value_template: "{{ state_attr('sensor.linha_do_metro', 'resposta').verde }}"
              tipo_msg_vd:
                  friendly_name: "Tipo de mensagem verde metro"
                  value_template: "{{ state_attr('sensor.linha_do_metro', 'resposta').tipo_msg_vd }}"
