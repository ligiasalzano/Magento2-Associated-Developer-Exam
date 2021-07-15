---
title: Processamento do Fluxo de Requisição
description: Processamento do Fluxo de Requisição no Magento 2
permalink: /processamento-do-fluxo-de-requisicao
---

1. Descrever como usar os modos Magento
2. Demonstrar a capacidade de criar um controller frontend com diferentes tipos de resposta (HTML/JSON/redirect)
3. Demonstrar como usar reescritas de URL de uma página de produto do catálogo para um URL diferente
{:toc}

## Descrever como usar os modos Magento

**Entender os prós e contras de usar o modo de desenvolvedor ou o modo de produção.**

O Magento possui três modos: o _default_, o _developer_ e o _production_. ([Referência](https://devdocs.magento.com/guides/v2.4/config-guide/bootstrap/magento-modes.html)).

- **_Default mode_**: 
  - É um híbrido dos modos _developer_ e _production_. Por isso não é o modo ideal nem para desenvolver e nem para a loja em produção.
  - Ele é o modo padrão, já vem definido na instalação do Magento
  - Utiliza links simbólicos
  - Erros não são mostrados (eles são colocados nos arquivos de log)
  - Arquivos estáticos são gerados em tempo real e _symlinked_ na pasta `var/view_prepeocessed`.
- **_Developer mode_**:
  - Ideal para desenvolvimento.
  - Baixa performace (bem mais lento).
  - Utiliza links simbólicos. Eles podem ser atualizados com o comando `bin/magento setup:static-content:deploy -f` ou removidos manualmente.
  - Os erros são mostrados ao usuário e o log é detalhado. Observação: o log de depuração está desativado por padrão, mesmo no modo de desenvolvedor.
  - Não deve ser usado em produção pois apresenta riscos de segurança.
- **_Production mode_**
  - Ideal para produção.
  - Melhor performace (mais rápido).
  - Não usa links simbólicos.
  - Erros são salvos nos arquivos de log.
  - Arquivos estáticos são pré-compilados, a compilação não acontece em tempo real.
- **_Maintenance mode_**
  - Redireciona os usuários para uma página de "Serviço Temporariamente Indisponível"
  - Quando este modo está ativo, o arquivo `.maintenance.flag` é criado na pasta `var/`.
  - É possível configurar uma lista de IPs como exceção ao modo de manutenção

**Como você ativa/desativa o modo de manutenção?**

Quando o modo de manutenção está ativo, o arquivo `.maintenance.flag` é criado dentro da pasta `var/`. Quando este arquivo não existe, a loja funciona normalmente.
Há também o arquivo `.maintenance.ip`, no mesmo diretório (`var/`). Nele existe uma lista de IPs que são exceção ao modo de manutenção.

Comandos:
- Habilitar o modo manutenção: `bin/magento maintenance:enable [--ip=<ip address> ... --ip=<ip address>] | [ip=none]` 
- Desabilitar: `bin/magento maintenance:disable [--ip=<ip address> ... --ip=<ip address>] | [ip=none]`
- Verificar o status: `bin/magento maintenance:status`

Onde:
- `--ip=<ip address>` representa um IP que será exceção ao modo de manutenção

Para salvar uma lista de IPs, você pode usar: `bin/magento maintenance:allow-ips <ip address> .. <ip address> [--none]`. 
O `--none` limpa a lista.

> Depois de colocar o Magento no modo de manutenção, você deve parar todos os processos do consumidor da fila de mensagens.
> Uma maneira de encontrar esses processos é executar o comando `ps -ef | grep queue:consumer:start`. E então executar o comando `kill <process_id>` para cada consumidor. Em um ambiente com vários nós, certifique-se de repetir esta tarefa em cada nó.


## Demonstrar a capacidade de criar um controller frontend com diferentes tipos de resposta (HTML/JSON/redirect)

**Como você identifica qual módulo/controller corresponde para um determinado URL?**

O Magento determina a área baseado no _frontname_ (`\Magento\Framework\App\AreaList::getCodeByFrontName`). Se nenhum _frontname_ corresponder, a área do _frontname_ padrão é carregada.
Se a solicitação não for para a API, o Magento analisa o URL. Essa operação é tratada pelo `\Magento\Framework\App\Router\Base::parseRequest`. O caminho (o segmento após o domínio) é explodido com a barra como delimitador.

O formato de uma url é esse: `sualoja.com/nome_da_rota/nome_do_controller/action`.

Exemplo: sualoja.com/catalog/product/view/id/42
- Primeira parte: **_frontname_** é o primeiro parâmetro. Nesse caso é o _catalog_. Ele é configurado em `etc/[area]/routes.xml`. 
- Segunda parte: é o caminho para a parte que contém o arquivo do _contoller_. É onde está a ação. Nesse caso, dentro do caminho `Controller/Product/`.
- Terceira parte: é o nome da _action_. Nesse caso, o Magento produra o arquivo `View.php` e roda a método `execute()` dentro dele.
- O final: `/id/42` corresponde à um parâmetro. É o mesmo que `id=42`.


**O que você faria para criar um determinado URL?**
O URL pode ser criado de várias formas.
- Definindo o atributo `url_key` para categorias e produtos.
- Criando um _URL rewrite_ em `Marketing > SEO & Search > URL Rewrites`
- Dentro de um módulo, criando o frontname no arquivo `My_Company/My_Module/etc/[area]/routes.xml` e o _controller_ em `My_Company/My_Module/Controller/MyController/ActionFile.php`. Observação: todos os _Controllers_ devem estender `\Magento\Framework\App\Action\Action` e ter um método `execute()`.


## Demonstrar como usar reescritas de URL de uma página de produto do catálogo para um URL diferente

**Como o URL amigável de um produto ou categoria é definido?**

Através do atributo `url_key`. Se ele não for definido, ele automaticamente corresponderá ao slug do produto ou categoria.

**Como você pode mudá-lo?**

Editando a `url_key` na página de edição do produto ou categoria.

**Como você determina qual página corresponde a um determinado URL amigável?**

Através da reescrita de URL ou da criação de uma nova rota em um módulo personalizado.

Na tabela _url_rewrite_, você encontrará uma linha em que o valor _request_path_ é o URL amigável. O valor _target_path_ correspondente é a página interna do Magento. O módulo _Magento_UrlRewrite_ contém um _router_ que verifica se o URL fornecido pode ser correspondido a um _request_path_ na tabela _url_rewrite_, redirecionando para o _target_path_ se uma correspondência for encontrada.
