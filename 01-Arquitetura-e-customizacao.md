---
title: Arquitetura Magento e Técnicas de Customização
description: Arquitetura e técnicas de customização do Magento 2
permalink: /arquitetura-e-customizacao
---

1. Descrever a arquitetura baseada em módulos da Magento
2. Descrever a estrutura de diretórios da Magento
3. Utilizar o escopo de configuração e variáveis de configuração
4. Demonstrar como usar a injeção de dependência (Dependency Injection - DI)
5. Demonstar habilidade no uso de plugins
6. Configurar event observers e trabalhos agendados (scheduled jobs)
7. Utilizar o CLI
8. Descrever como as extensões são instaladas e configuradas
{:toc}

## Descrever a arquitetura baseada em módulos do Magento

Os módulos Magento 2 utilizam a arquitetura MVVM. Que tem 3 camadas:
- Model: Lógica de negócio e contado com a database.
- View: Estrutura e layout
- ViewModel: Liga as camadas Model e View

Em seu nível mais alto, a arquitetura do produto Magento consiste no código do produto principal mais módulos opcionais. Esses módulos opcionais aprimoram ou substituem o código básico do produto.
[A arquitetura do Magento 2 consiste em 4 camadas](https://devdocs.magento.com/guides/v2.4/architecture/archi_perspectives/ALayers_intro.html):
- Presentation Layer: Contém _view elements_, como layouts, blocos, _templates_ e _controllers_
- Service Layer: Intermediário entre as camadas _Presentation_ e _Domain_. Executa os contratos de serviço (que permitem adicionar ou modificar a regra de negócio).
- Domain Layer: Responsável pela lógica de negócio.
- Persistence Layer: Descreve um _resource model_ que é responsável por recuperar e modificar dados na _database_ usando CRUD.

### Descrever a arquitetura do módulo.

Um módulo é um **grupo lógico** – ou seja, um diretório contendo _blocks, controllers, helpers e models_ – que é relacionado à um **recurso específico** do negócio. Cada módulo pode adicionar um novo recurso implementando uma nova funcionalidade ou estendendo a funcionalidade de outro módulo. Contudo, o Magento possui foco na modularidade, por isso, um módulo deve trazer apenas um recurso e depender minimamente de outros módulos.

Dessa forma, um módulo deverá ser o mais independente e específico possível. Sendo que a inclusão ou exclusão de um módulo não deve afetar a funcionalidade de outros. [Magento DevDocs - Module overview](https://devdocs.magento.com/guides/v2.4/architecture/archi_perspectives/components/modules/mod_intro.html)

**As áreas do Magento**

Há 7 áreas no Magento: ([referência](https://devdocs.magento.com/guides/v2.4/architecture/archi_perspectives/components/modules/mod_and_areas.html#magento-area-types))

- `adminhtml` - A área adminhtml inclui o código necessário para o gerenciamento da loja. Refere-se ao painel administrativo (admin ou backend) do Magento. 
- `frontend` - Refere-se à parte da loja na qual haverá a interação com o cliente (usuário final)
- `global` (ou `base`) - Engloba todas as áreas e é usado como _fallback_ para arquivos ausentes nas áreas `frontend` e `adminhtml`
- `crontab` - É carregada quando as _crons_ são executadas. A classe `\Magento\Framework\App\Cron` sempre carrega a área `crontab`.
- `webapi_rest` - Esta área responde pelas chamadas _REST_ da _API_ do Magneto 2.
- `webapi_soap` – Refere-se às chamadas _SOAP_.
- `graphql` - Responde pelas chamadas GRAPHQL (essa área foi inserida a partir do Magento 2.3)

Nem todas as áreas estão disponíveis todo o tempo. Por exemplo, a `crontab` é usada apenas ao executar tarefas cron.

###  Quais são os principais passos para adicionar um novo módulo?

Os arquivos necessários para inicializar um módulo são: `registration.php`, `etc/module.xml` e `composer.json`. Sendo que são obrigatórios o `registration.php` e o `etc/module.xml`.

**1º - Crie a estrutura de diretório básica do seu módulo:**
Por exemplo, dentro de `/app/code`, crie o diretório `SuaEmpresa/SeuModulo`.

**2º - Declare o componente e suas dependências no `composer.json`.**

Nesse arquivo, informamos o nome, descrição, autor e versão do módulo, suas dependências e outras informações.
No `composer.json`, o **_autoload_** especifica as informações necessarias para serem carregadas, como o arquivo _registration.php_.

```json
{
 "name": "suaempresa/seu-modulo",
 "autoload": {
    "psr-4": { "SuaEmpresa\\SeuModulo\\": "" },
    "files": [ "registration.php" ]
 }
}
```

**3º - Registre o componente com o arquivo `registration.php` (obrigatório).**

Este arquivo é incluído pelo _Composer autoloader_ (`app/etc/NonComposerComponentRegistration.php`).
Isto adiciona o módulo à lista de componentes em `\Magento\Framework\Component\ComponetRegistrar`.

Após a leitura deste arquivo, o Magento vai procurar o `etc/module.xml`.

**[Registrando alguns tipos de componentes](https://devdocs.magento.com/guides/v2.4/extension-dev-guide/build/component-registration.html)**
- **Módulos:** `ComponentRegistrar::register(ComponentRegistrar::MODULE, '<VendorName_ModuleName>', __DIR__);`
- **Temas:** `ComponentRegistrar::register(ComponentRegistrar::THEME, '<area>/<vendor>/<theme_name>', __DIR__);`
- **Pacotes de tradução:** `ComponentRegistrar::register(ComponentRegistrar::LANGUAGE, '<VendorName>_<packageName>', __DIR__);`
- **Bibliotecas:** `ComponentRegistrar::register(ComponentRegistrar::LIBRARY, '<vendor>/<library_name>', __DIR__);`


**4º - Nomeie, declare e defina as dependências no arquivo module.xml (obrigatório).**

Cada módulo deve ser nomeado e declarado em um arquivo xml específico do componente. 
- module.xml (modules), theme.xml (themes) and language.xml (for language packages).

O arquivo module.xml é localizado no diretório `<vendor>/<module>/etc`. Neste arquivo também são declaradas a versão do módulo (que será utilizada pelas classes de Setup do módulo, se houverem) e as suas dependências.

```xml
<?xml version="1.0"?>
<config xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="urn:magento:framework:Module/etc/module.xsd">
    <module name="SuaEmpresa_SeuModulo" setup_version="1.0.0" />
</config>

```

As dependências são carregadas no elemento `<sequence>` (que pertence ao nó `<module>`). Aqui também é determinada a sequência de carregamento:
```xml
<sequence>
  <module name="Vendor_Module"/>
</sequence>
```
O nó `<sequence>` reporta ao Magento em qual ordem carregar os módulos. Esse ajuste acontece quando o módulo é instalado. A lista de módulos é armazenada no arquivo `app/etc/config.php`. Esta sequência é montada em `vendor/magento/framework/Module/ModuleList/Loader.php`. Esta organização é muito útil para _events_, _plugins_, _preferences_ e _layouts_.

Na documentação oficial do Magento, temos um exemplo de como criar um novo módulo. [Acesse por aqui](https://devdocs.magento.com/videos/fundamentals/create-a-new-module/).

**Dependencias do módulo**

No Magento 2 os módulos possuem dois tipos de dependências: _hard (forte)_ e _soft (fraca)_. Módulos com dependência forte não funcionam sem o módulo do qual dependem. Já os de depêndencia fraca funcionam independentemente de outro módulo.

**_Hard dependencies_** são definidas na seção `require` do arquivo `app/code/<Vendor>/<Module>/composer.json`.

**_Soft dependencies_** são definidas na seção `suggest`do arquivo `app/code/<Vendor>/<Module>/composer.json`.
Elas também estão no nó `<sequence>` do `app/code/<Vendor>/<Module>/etc/module.xml`.

Dentro dos tipos de depências, um módulo pode ter vários tipos de relações com outros módulos. Como:
> - **uses:** módulo A usa o módulo B;
> - **reacts to:** o módulo A reage ao módulo B se o seu comportamento for acionado por um evento que ocorre no módulo B (isso acontece sem o "conhecimento" do módulo B);
> - **customizes:** o módulo A personaliza o módulo B;
> - **implements:** o módulo A implementa o módulo B;
> - **replaces:** o módulo A substitui o módulo B;

> ([Referência](https://amasty.com/blog/magento-2-certification-module-based-architecture/))

Um dos princípios do Magento é a **minimização de dependências**, ou seja, um módulo deve ter o mínimo possível de dependências, bem como, não depender de vários módulos. Isso tem relação com o **princípio de responsabilidade única** em que o módulo deve ter apenas uma função.

###  Quais são os diferentes tipos de pacotes do Composer? 

**Tipos de componentes**

No Magento, [existem 6 tipos de componentes](https://devdocs.magento.com/guides/v2.4/extension-dev-guide/prepare/dev-modtypes.html):
- Módulos (_magento2-module_): Adicionam ou estendem uma funcionalidade.
- Temas (_magento2-theme_): Personalizam a aparência. Podem ser para o frontend ou para a área adminhtml.
- Pacotes de tradução (_magento2-language_): Adiciona traduções de idioma.
- Bibliotecas (_magento2-library_): Suporte para blibliotecas localizadas em `./lib/internal/`.
- Metapackages (_metapackage_): Tecnicamente, é um tipo de pacote do composer e não no Magento. Consiste em um arquivo `composer.json` que especifica uma lista de componentes e suas dependências. Tanto o Magento Open Source quanto o Magento Commerce são metapackages.
- Componentes (_magento2-component_): Pacote formado pelos arquivos que devem estar localizados na raiz (.htaccess, etc). Isso inclui `dev/tests` e setup.


Use o seguinte formato quando for nomear o seu pacote: `<vendor-name>/<package-name>`

`vendor-name`: Todas as letras devem ser minúsculas. Por exemplo, o nome _vendor_ para extensões desenvolvidas pela Magento Inc é _magento_.

`package-name`: Todas as letras devem ser minúsculas. A convenção para o nome dos pacotes da Magento é: `magento/<type-prefix>-<suffix>`

`<type-prefix>` é qualquer tipo de extenção do Mangento. Pode ser, por exemplo: _module-_ (para módulos), _theme-_ (para temas), _language-_ (para pacotes de tradução), _product-_ (para **_metapackages_** como o _Magento Open Source_ ou _Magento Commerce_.


###  Quando você deve colocar um módulo no app/code versus outros locais?

Os módulos estão localizados nos diretórios **vendor** e **app/code**. 
- No diretório **vendor** são colocados os módulos instalados via Composer. Ali também estão os módulos principais do Magento (caso ele tenha sido instalado via Composer). Os arquivos que estão nesta pasta, são considerados "intocáveis", ou melhor, não podem ser editados diretamente. Os módulos deste diretório seguem a seguinte síntaxe: `vendor/<vendor>/<type>-<module-name>`, onde `<type>` pode ser _module_, _theme_ ou _language_.
- O diretório **app/code** é recomendado para o desenvolvimento de módulos (ou instalações de módulos sem ser pelo Composer). Aqui, o diretório do módulo fica assim: `app/code/<vendor>/<module-name>`. 


## Descrever a estrutura de diretórios do Magento

### Descrever a estrutura de diretórios do Magento

Toda a estrutura do Magento pode ser dividida nos seguintes tipos:
- Estrutura da instalação Magento (root)
- Estrutura dos temas
- Estrutura dos módulos

#### Estrutura de arquivos da intalação Magento

**./app/**

Contém módulos, temas, traduções e configurações globais.
- `./app/code/`: módulos
- `./app/design/`: temas
- `./app/etc/`: configurações globais
- `./app/i18n/`: traduções

**./bin/**

O script executável da CLI do Magento. Daqui vem o comando `bin/magento`.

**./dev/**

Ferramentas para desenvolvedores. Armazena testes funcionais automatizados que foram executados pelo Magento Test Framework.

**./generated/**

Aqui é armazenado os códigos gerados automaticamente pelo Magento, como _Factories_, _Proxies_, _Interceptors_ etc. Na configuração padrão, se a classe for injetada em um construtor, o código será gerado pelo Magento para criar _non-existent factory classes_. 

**./lib/**

Este diretório contém todos os arquivos de bibliotecas do Magento e do `vendor`. Ele também inclui todo o código Magento não baseado em módulo.

**./phpserver/**

Aqui fica o arquivo `Router.php`, o qual é usado para implementar o servidor _built-in_ do PHP. No entanto, por questões de segurança, não é recomendável trabalhar com ele. Serve para casos específicos.

**./pub/**

Inclui os arquivos públicos do Magento, como arquivos estáticos (css, imagens, js etc) e páginas de erro. Quando em produção, o Magento deve apontar para esta pasta. Isto tras segurança à aplicação por proteger a raiz da instalação. 
- `pub/errors` - erros
- `pub/media` - imagens
- `pub/static` - arquivos gerados pelo Magento
- `pub/cron.php`
- `pub/index.php` - primeiro arquivo carregado pela aplicação

**./setup/**

Aqui tem os arquivos do instalador do Magento. 
> A partir da versão 2.4 do Magento, só é possível instalá-lo através da cli.

**./var/**

Arquivos temporários/gerados como cache, logs, report, sessão, erros e alguns arquivos compilados
- `var/log`: arquivos de log do sistema, como `exception.log` e `system.log`.
- `var/cache`: contém todo o cache do Magento. Esta pasta é limpa quando rodamos o `bin/magento cache:clean`
- `var/view_preprocessed`: contém arquivos style gerados e HTML minificado

**./vendor/**

Diretório nativo do _Composer_. Contém o _framework core_. Tudo o que for instalado pelo _Composer_ vem para este diretório.
Aqui encontram-se todos os módulos do Magento em si.


#### Estrutura de arquivos dos temas

**/etc**

Arquivos de configuração como o view.xml

**/i18n**

Dicionários de tradução do tema

**/media**

Imagens de pré-visualização do tema.

**/web**

Arquivos estáticos.

- `web/css/source`: arquivos de configuração _LESS_ do tema
- `web/css/source/lib`: Inclui arquivos de exibição que substituem os arquivos de biblioteca da UI armazenados em `lib/web/css/source/lib`
- `web/fonts`: arquivos de fonte
- `web/images`: imagens
- `web/js`: arquivos JavaScript

#### Estrutura de arquivos dos módulos

**/Api: Service Contracts - Contratos de serviço**

Aqui ficam todas as interfaces responsáveis pelos contratos de serviço (_services contracts_). Contém classes que serão expostas na api do Magento. Também contém métodos que podem ser reutilizados em outros locais da aplicação. Exemplo `\Magento\Catalog\Api\CategoryListInterface`.

**/Api/Data: Data Service Contracts - Dados dos contratos de serviço**

Esta pasta contém interfaces que representam dados. Exemplos: _Product interface, Category interface_ e _Customer interface_.

**/Block: View Models**

Este diretório faz parte da camada View do MVC. Contém os _View Models_ para os templates do Magento. Os _blocks_ são as classes responsáveis por fazer a inteface entre o template e os _resource models_ do Magento para obter dados, trabalhá-los e passá-los para o template. Eles fornecem a lógica de negócio para os templates, que devem usar o mínimo de PHP (separação de responsabilidades). Veja mais [aqui](https://stackoverflow.com/a/4946409) e [aqui](https://code.tutsplus.com/articles/improving-your-work-flow-separate-your-mark-up-from-your-logic--wp-29180).

**/ViewModel**
Contém classes que auxiliam os blocos no fornecimento de informações para os templates.
A fim de evitar que os contrutores dos Blocos ficam muito grandes (tornando os testes e reutilizações difíceis), separamos as responsabilidades. Enquanto Blocos tratam da renderização dos blocos, os View Models cuidam da lógica de negócios.

Para fornecer um View Model para um bloco, informamos através de um argumento na declaração do bloco dentro do xml de layout:

```xml
<block name="product.promotions" template="Vendor_Module::template.phtml" >
    <arguments>
        <argument name="viewModel" xsi:type="object">Vendor\Module\Block\ViewModelClass</argument>
    </arguments>
</block>
```

Leia mais sobre no artigo [Better Blocks: Magento 2 PHP View Models](https://firegento.com/blog/2017/12/07/better-blocks-magento-2-php-view-models/).

**/Console: Console Commands**

Abriga os códigos para os comandos do `bin/magento`. Cada comando que aparece na listagem do CLI é referente à uma classe desse diretório.

**/Controller: Web Request Handlers**

Todos os _controllers_ do módulo ficam aqui. Cada _controller_ deve ter uma única responsabilidade (uma _Action_). Quando uma página é requisitada, o caminho é construído com parâmetros do arquivo `routes.xml` e dos _controllers_ do diretório `/Controller`.

**/Controller/Adminhtml: Admin controllers**

Aqui ficam os _controllers_ da área adminhtml.

**/Cron: Cron Job Classes**

Possui as crontabs que serão executadas no Magento (tarefas agendadas). 

**/etc: Configuration files**

Este diretório engloba todos os arquivos `.xml` de configuração do módulo. Aqui, as configurações podem ser globais (`etc/`) ou por área (`etc/frontend` ou `etc/adminhtml`).
Alguns arquivos precisam estar dentro de uma área (`routes.xml` e `sections.xml`), outros devem ser globais (`acl.xml`) e outros podem ser globlais ou específicos de área (`di.xml`).

**/Helper: Occasionally useful for small, reusable code**

Classes que contém métodos auxiliares (métodos estáticos), que não devem depender de outras classes.

**/i18n: Translation CSV Files**

Aqui estão todos os arquivos CSV de tradução. Esses CSVs possuem duas colunas: _de_ e _para_.

**/Model: Data Handling and Structures**

Nesta pasta ficam os _Models_, classes que lidam diretamente com o banco de dados. 

**/Model/ResourceModel: Database Interactions**

Aqui são as classes que definem como os dados devem ser recuperados e salvos no banco de dados. Qualquer interação direta com o bando de dados deve acontecer nesses arquivos.

**/Observer: Event Listeners**

Todos os _Observers_ são incluídos neste diretório. Quando o Magento dispara um evento, as classes que estão observando esse evento são chamadas. Os _event listeners_ devem implementar o `ObserverInteface` (`\Magento\
Framework\Event\ObserverInterface`). A classe PHP deve serguir o padrão _TitleCase_ enquanto o evento deve seguir o _snake_case_. 
Deve-se evitar colocar lógica de negócio em um observer. O correto seria injetá-la no _Observer_. 

**/Plugin: Function Modification**

O conceito de _Plugin_ foi introduzido no Magento 2. Os _Plugins_ permitem modificar funcionalidades da maioria das classes e interfaces. Plugins funcionam apenas em objetos que foram instanciados pelo _ObjectManager_. Por convenção, os plugins são colocados no diretório `/Plugin`.

**/Setup: Database Modification**

Os arquivos que modificam o banco de dados durante a instalação ou atualização de um módulo são colocados neste diretório. 
Os arquivos utilizados aqui são:

- `InstallSchema.php`: configura o esquema de tabelas e/ou colunas na ocasião da instalação do módulo.
- `UpgradeSchema.php`: modifica as tabelas e colunas quando a versão do módulo é atualizada.
- `Recurring.php`: É executado depois de todas as instalações ou atualizações.
- `InstallData.php`: Configura dados quando o módulo é instalado.
- `UpgradeData.php`: Modifica dados quando o módulo é atualizado.
- `RecurringData.php`: aplica-se aos dados após cada instalação ou atualização.

> Para modificar o banco de dados, o Magento criou o `db_schema.xml` que deverá substituir os scripts do diretório _Setup_.

**/Test:**

O Magento 2 adotou o TDD (Test Driven Development). Todos os testes ficam neste diretório.

**/Ui: UI Component Data Providers**

Aqui ficam todos os provedores e modificadores de dados utilizados pelos componentes UI.

**/view/[area]/layout: Layout XML**

Aqui temos os arquivos .xml que estruturam o layout e ligam os blocos com os templates. O nome dos arquivos são formados da seguite forma: `{route_id}_{controller}_{action}.xml`. 

**/view/[area]/templates: Block Templates**

Enquanto a lógica de negócio é representada nos `Blocks`, a forma como essa lógica é apresentada ao usuário é definida nos arquivos de template. Esses arquivos devem conter o mínimo possível de PHP. 
> Observação: note que o nome desta pasta é escrito no plural: templates.

**/view/[area]/page_layout**

Os arquivos que descrevem o esqueleto do layout (1column.xml, 2columns-left.xml...)

**/view/adminhtml/ui_component: UI Components**

Este diretório contêm os arquivos XML de configuração dos _UI Components_. Eles flexibilizam a renderização da interface do usuário. _Grids_ e _Forms_ do admin, bem como o _checkout_ são feitos com _UI Components_.

**/view/[area]/web: Web Assets**

Este é o lugar onde os recursos da Web são armazenados. Isso inclui JS, CSS (ou LESS / SCSS) e imagens.

**/view/[area]/web/template: JS Templates**

Os modelos HTML que podem ser solicitados de forma assíncrona com Javascript são colocados aqui.
Observação: cuidado para não confundir. O nome desta pasta é no singular, template.

**/view/[area]/requirejs-config.js**

Aqui ficam as configurações RequireJS do módulo. Essa configuração é usada para controlar as dependências do módulo Javascript, criar aliases e declarar mixins

**Quais são as convenções de nome e como os _namespaces_ são estabelecidos?**

Os _namespaces_ são utilizados para evitar conflitos de nomes no código. Eles ajudam a deixar as coisas organizadas. No PHP, um _namespace_ determina onde o arquivo PHP (o arquivo da classe) está localizado na hierarquia do _namespace_. Assim, o nome da classe pode ser reutilizado em _namespaces_ diferentes. 
Além disso, O _namespace_ e o nome da classe auxiliam para identificar onde está o arquivo.

O _namespace_ no Magento é composto desta forma: `Vendor/Module/Path/Class`. Por exemplo, a classe `RegistryConstants`, pertencente ao módulo `Magento_CatalogRule`, pertence ao _namespace_ `Magento\CatalogRule\Controller`. Isso significa que o arquivo da classe (`RegistryConstants.php`) fica dentro de: `/vendor/magento/module-catalogrule/Controller/RegistryConstants.php` ou `/app/code/Magento/CatalogRule/Controller` (dependendo do tipo da instalação).

**Semântica**

1. Para nomes de atributos e valores, deve-se utilizar palavras escritas em minúsculo, não abreviadas, com caracteres do Latin e concatenadas com hífen (-)
Exemplo:

```html
<section id="information-dialog-tree">
   <p> ... </p>
   <p> ... </p>
</section>
<a href="#information-dialog-tree">Scroll to text</a></a>
```

**Como identificar os arquivos responsáveis por uma certa funcionalidade?**
Alguns diretórios já estão definidos, por convenção, para serem responsáves por certas funcionalidades. Eles foram descritos nesta seção. Contudo, um módulo pode conter um diretório específico para alguma funcionalidade pouco comum. Nestes casos, é necessário inspecionar o diretório para averiguar a funcionalidade pela qual ele é responsável.

Para encontrar os arquivos responsáveis por certa funcionalidade, podemos procurar nos arquivos `di.xml`. No caso de arquivos de template ou classes Block, podemos encontrá-los através das Template Hints que podem ser habilitadas nas configurações da loja, no admim, em: Lojas > Configuração > Avançado > Desenvolvedor.
Para encontrar actions, podemos investigar o arquivo `routes.xml` e buscar o _controller_ que costuma estar em `<module_dir>/Controllers/<Controller>/<Action>`.
Podemos ainda fazer uma busca por um conteúdo específico, uma string, através do `grep`.

## Utilizar o escopo de configuração e o escopo de variáveis de configuração

### Determinar como usar os arquivos de configuração na Magento. Quais arquivos de configuração são importantes no ciclo de desenvolvimento?

O Magento 2 dividiu as configurações em vários arquivos XML, isso evita que tenhamos apenas um arquivo muito longo. Adicionalmente, podemos separar os arquivos em áreas, restringindo determinada configuração à área a qual ela afeta.
Logo abaixo, estão relacionados os principais arquivos de configuração do Magento. Eles estão localizados no diretório `/etc` do módulo.

**`module.xml`**

É o único arquivo de configuração obrigatório. Nele são especificados a versão do módulo e sua ordem de carregamento.

**`acl.xml`**

_Access Control List Rules (ACL)_. Aqui são definidas as permissões de acesso dos usuários à recursos. 

**`config.xml`**

Carregado na configuração padrão (`Stores > Configuration`). 

**`crontab.xml`**

Para os trabalhos agendados.

**`di.xml`**

É um dos arquivos mais utilizados quando estamos personalizando o Magento. Aqui são configuradas as injeções de dependência: plugins são definidos, classes são substituídas, classes concretas são especificadas, dentre outros.

**`email_templates.xml`**

Determina os templates de e-mail que serão usados no Magento.

**`events.xml`**

Esse arquivo frequentemente é criado dentro de uma área específica. Nele são registrados os _event listeners_.

**`indexer.xml`**

Configura os indexadores do Magento.

**`adminhtml/menu.xml`**

Configura um menu na área adminhtml.

**`adminhtml/system.xml`**

Aqui são definidas as _tabs_, _sections_, _groups_ e _fields_ que existem na configuração da loja (`Stores > Configuration`).

**`mview.xml`**

Usado frequentemente para o processo de indexação, aqui é configurado o disparo de um evento quando uma coluna do banco de dados é alterada.

**`<area>/routes.xml`**

Diz ao Magento que a área aceita _web requests_. O nó da rota configura a primeira parte do _layout handle_ (ID da rota) e o _frontname_ (primeira parte da URL após o domínio).

**`view.xml`**

Especifica valores padrão para configurações relacionadas ao design. É semelhante ao `config.xml`.

**`webapi.xml`**

Configura a web API. 

**`widget.xml`**

Configura os widgets para serem usados com páginas ou blocos CMS e produtos.

> Mais arquivos de configuração: [DevDocs - Module configuration files](https://devdocs.magento.com/guides/v2.4/config-guide/config/config-files.html#config-files-classes-objects)


### Descrever o desenvolvimento no contexto dos escopos website e store. 

**Como você pode identificar o escopo de configuração para uma certa variável?**

Quando criamos campos de configuração no Magento, através do arquivo `etc/adminhtml/system.xml`, podemos determinar o escopo daquela informação no próprio nó, através dos argumentos `showInDefault="1"`, `showInWebsite="1"` e `showInStore="1"`. Veja abaixo um exemplo de um _field_, para mais detalhes, consulte a [documentação](https://devdocs.magento.com/guides/v2.4/config-guide/prod/config-reference-systemxml.html).
```xml
<field id="UNIQUE_FIELD_ID" translate="label" sortOrder="10" showInDefault="1" showInWebsite="1" showInStore="1" type="text">
    <label>A meaningful field label</label>
    <comment>A descriptive text explaining this configuration field.</comment>
    <tooltip>Another possible frontend element that also can be used to describe the meaning of this field. Will be displayed as a small icon beside the field.</tooltip>
    <validate>required-entry no-whitespace</validate> <!-- Field is required and must not contain any whitespace. -->
    <if_module_enabled>VENDOR_MODULE</if_module_enabled>
    <depends> <!-- This field will only be visible if the field with the id A_UNIQUE_FIELD_ID is set to value 1 -->
       <field id="A_UNIQUE_FIELD_ID">1</field>
    </depends>
</field>
```

**Como os escopos nativos da Magento (por exemplo, preço ou estoque) podem afetar o desenvolvimento e o processo de tomada de decisão?**

Com os escopos, podemos ter um Magento com vários websites e, dentro de cada website, várias lojas. Antes de tudo devemos entender as necessidades do cliente para planejar se o desenvolvimento deve ocorrer com um website e várias lojas, ou com vários websites e uma loja em cada um ou, ainda, vários websites e várias lojas. O processo de tomada de decisão deve considerar a necessidade de compartilhamento (ou isolamento) do catálogo de produtos ou base de clientes entre estes websites/lojas e definições sobre métodos de pagamento, moeda, estoque, preço e idioma, entre outros. 
Para saber mais sobre como funcionam os escopos, consulte [este artigo](https://www.magenteiro.com/blog/para-magenteiros/multiloja-com-magento/?mid=c9f0f895fb98ab9159f51fd0297e236d) e a [documentação oficial](https://docs.magento.com/user-guide/configuration/scope.html).

### Demonstrar capacidade de adicionar valores diferentes para diferentes escopos.

**Como você pode buscar o valor de uma configuração do sistema por meio de programação?**

Você pode usar os métodos contidos na classe `Magento\Framework\App\Config\ScopeConfigInterface`, passando o caminho da configuração como parâmetro, por exemplo: `web/secure/base_url`.

```php
$this->scopeConfig->getValue(static::PATH_ENABLED, $scopeType, $scopeCode);
```

**Como você pode substituir os valores de uma configuração do sistema para uma determinada loja usando a configuração XML?**

Você pode definir valores de configuração através do arquivo `config.xml`.

## Demonstrar como usar a injeção de dependência (DI)

> A injeção de dependência (**Dependency Injection - DI**) é uma forma de dar à uma classe o que ela precisa para funcionar. 

A injeção de dependência é um padrão de design que permite declarar dependências de objetos no Magento 2. A utilização da DI permite desenvolver um código mais estrutural e independente tornando o processo de codificação mais conveniente.
O Magento usa a injeção de construtor, todas as dependências são especificadas como argumentos na função `__construct()`.

**ObjectManager**
O `ObjectManager` é a unidade interna de armazenamento de objetos do Magento e raramente deve ser acessada diretamente. Ele torna possível a implementação do princípio de composição sobre herança.
Na [documentação do Magento](https://devdocs.magento.com/guides/v2.4/extension-dev-guide/depend-inj.html#object-manager), a definição de ObjectManager é: "uma classe de serviço do Magento que instancia objetos no início do processo de _bootstrapping_".

> O Magento proíbe o uso direto do ObjectManager no seu código porque oculta as dependências reais de uma classe. Veja as [regras de uso](https://devdocs.magento.com/guides/v2.4/extension-dev-guide/object-manager.html#usage-rules).

Responsabilidades do `ObjectManager`:
- Criação de objetos _factories_ e _proxies_
- Instanciar automaticamente parâmetros em construtores de classe
- Gerenciar dependências instanciando a classe de preferência quando um construtor solicita sua interface
- Implementar o padrão _singleton_ retornando a mesma instância compartilhada de uma classe quando solicitado

**Proxies**
As _proxies_ são usadas para a criação de objetos que demandam mais tempo para carregar. Nesses casos, uma classe _proxy_ lento carrega a classe. 
A _proxy_ é especificada no `di.xml` em uma classe como `\Magento\Catalog\Model\Product\Proxy`. 
> _Proxies_ não devem ser especificadas no método construtor.

**Factories**
Para objetos que precisam se criados todas as vezes que são usados, existem as _factories_. Para especificar uma _factory_, inclua a palavra _Factory_ no final da classe ou interface. Exemplo: `\Magento\Catalog\Api\Data\ProductInterfaceFactory`.


### Demonstrar a capacidade de usar o conceito de injeção de dependência no desenvolvimento Magento. 

A Injeção de Dependência é um design pattern que permite um objeto A declarar suas dependências para um objeto B que irá suprir estas dependências. As dependências declaradas em A são geralmente interfaces e as dependências fornecidas por B são implementações concretas para essas interfaces. [Saiba mais aqui](https://devdocs.magento.com/guides/v2.4/extension-dev-guide/depend-inj.html).

**Como os objetos são identificados no código?**

No Magento 1 os objetos são manipulados com o uso da classe Mage. 
No Magento 2, usamos a Injeção de Dependências (_Dependency Injection_), que substitui a funcionalidade fornecida pela classe Mage.

Como a injeção de dependência acontece automaticamente no construtor, o Magento deve lidar com a criação de classes. Como tal, a criação de classe acontece no momento da injeção ou com uma _factory_.

**Por que é importante ter um processo centralizado de criação de objetos?**
Ter um processo centralizado para criar objetos facilita muito o teste. Ele também fornece uma interface simples para substituir objetos e modificar os existentes.
É a criação centralizada de objetos que torna possível vantagens como:
- Usar a injeção de dependência automática permitindo que você faça injeções de código no código existente. Podemos criar variações da classe sem modificar a classe original. Ao usar o ObjectManager para criação de classes, as dependências são substituídas automaticamente no construtor da classe.
- Torna-se possível injetar as dependências da classe no construtor da classe por meio do arquivo di.xml.
- Permite evitar a herança, o que significa que os aplicativos se tornam mais flexíveis. Assim, você não precisa pensar nas classes filho ao alterar a classe pai.

### Identifique como usar arquivos de configuração DI para personalizar a plataforma Magento. 

Para fazer essas personalizações no Magento 2, você precisa usar o arquivo de configuração `di.xml`. Que pode ser global ou específico de uma área.

**Como substituir uma classe nativa**
> Se houver a possibilidade de usar _plugin_, evite sobreescrever uma classe. Isso pode gerar conflitos.

Para substituir uma classe nativa, use uma entrada `<preference />` para especificar o nome da classe existente (a barra invertida anterior \ é opcional) e a classe a ser substituída.
As preferências são usadas para substituir classes inteiras. Eles também podem ser usados para especificar uma classe concreta para uma interface.

```xml
<config>
    <preference for="Magento\Catalog\Api\Data\ProductInterface" type="YourVendor\Catalog\Model\Product" />
</config>
```

**Como injetar uma classe em outro objeto**
Use uma entrada `<type/>` com name `providers` (`<argument name="providers" xsi:type="object">\Path\To\Your\Class</argument>`) dentro do nó `<arguments/>`.

Exemplo: `Magento\Sales\Model\ResourceModel\Provider\UpdatedIdListProvider` é a nossa classe que queremos injetar e `vendor/magento/module-sales/Model/ResourceModel/Provider/NotSyncedDataProvider.php`é o objeto de classe no qual vamos injetar a nossa classe.

```xml
<type name="Magento\Sales\Model\ResourceModel\Provider\NotSyncedDataProvider">
 <arguments>
   <argument name="providers" xsi:type="array">
       <item name="default" xsi:type="string">Magento\Sales\Model\ResourceModel\Provider\UpdatedIdListProvider</item>
   </argument>
 </arguments>
</type>
```

**Virtual Types**

Um _Virtual Type_ permite que o desenvolvedor crie uma instância de uma classe existente que possui argumentos de construtor personalizados. Isso é útil nos casos em que você precisa de uma classe "nova" apenas porque os argumentos do construtor precisam ser alterados. Isso é usado frequentemente no Magento para reduzir classes PHP redundantes.

_Virtual Types_ são convenientes para DI apenas no caso de precisarmos indicar nossas classes nas dependências. Dessa forma, você não precisa criar uma classe separada.

```xml
<virtualType name="pdfConfigDataStorage" type="Magento\Framework\Config\Data">
    <arguments>
        <argument name="reader" xsi:type="object">Magento\Sales\Model\Order\Pdf\Config\Reader</argument>
        <argument name="cacheId" xsi:type="string">sales_pdf_config</argument>
    </arguments>
</virtualType>
<type name="Magento\Sales\Model\Order\Pdf\Config">
    <arguments>
        <argument name="dataStorage" xsi:type="object">pdfConfigDataStorage</argument>
    </arguments>
</type>
```

### Dado um cenário, determinar como obter um objeto usando o objeto ObjectManager. 

**Como você obteria uma instância de classe de diferentes locais no código?**

Podemos acessar uma classe de duas formas:

**Com PHP puro:**
Com ```php $object = new SomeClass();```.

**Com o ObjectManager (preferencialmente em vez do php puro):**
Usando o ```php $objectManager->create(‘SomeClass’);``` ou ```php $objectManager->get(‘SomeClass’);```.
O método _create_ instancia um novo objeto cada vez que é chamado. O método _get_ instancia um objeto uma vez e, em chamadas futuras, o _get_ retorna o mesmo objeto. Esse comportamento é semelhante às _factories_ `getModel` e `getSingleton` do Magento 1


## Demonstrar habilidade no uso de plugins 

### Demonstrar entendimento dos plugins. 

**Como os plugins são usados no código base? Como eles podem ser usados para personalizações?**

- Existem 3 tipos de plugins: _before_, _after_ e _around_. São úteis para modificar a entrada, saída ou execução de um método existente (cuidado com os plugins do tipo _around_).
- Plugins funcionam apenas em métodos públicos (não em privados ou protegidos)
- Plugins não funcionam em classes virtuais e finais, métodos finais, estáticos e não públicos, construtores e objetos instanciados antes da inicialização do `Magento\Framework\Interception`.
- Eles são configurados no `di.xml`
- Plugins podem ser usados em interfaces, classes abstratas ou classes pai. Os métodos de plugin serão chamados para qualquer implementação dessas abstrações.
- É bom evitar o uso de plugins em situações em que um _observer_ funcione.


## Configurar event observers e trabalhos agendados (scheduled jobs)

> _Observers_ e _scheduled jobs_ não devem ser usados para modificar dados. Para isso, use plugins.

### Demonstrar como criar uma customização usando um event observer. 

**Como os observers são registrados? Como eles são definidos para frontend ou backend?**

Os _observers_ são registrados no arquivo `events.xml`, no diretório `/etc`. Se o evento for específico para uma área (backend ou frontend), é criado um diretório para a área requerida. 
Ex.: `/etc/[area]/events.xml` - `/etc/frontend/` e `/etc/adminhtml/`.

O evento é registrado no nó `<event>`:

```xml
<event name="event_for_your_observer_to_listen_for">
   <observer name="observer_name" instance="Your\Observer\Class" />
</event>
```

**Como os eventos automáticos são criados e como eles devem ser usados?**
Eventos devem ser usados quando não se quer modificar dados. Eles podem ser acionados injetando uma instância do `Magento\Framework\Event\ManagerInterface` no construtor e chamando: ```php $this->eventManager->dispatch('event_name', [params]);```.

**Como os trabalhos agendados (_scheduled jobs_) são configurados?**
Os trabalhos agendados são configurados no arquivo `crontab.xml`, dentro do diretório `/etc` (não em `/etc/[area]`).
Para configurar um trabalho agendado, é necessário atribuir um nome a ele, especificar a função que ele deve executar, a classe à qual essa função pertence e definir o agendamento usando a notação regular de agendamento cron.

```xml
<config xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"    
        xsi:noNamespaceSchemaLocation="urn:magento:module:Magento_Cron:etc/crontab.xsd">
    <group id="default">
        <job name="cron_job_name"
             instance="Path\To\Your\Class"
             method="execute">
            <schedule>*/5 * * * *</schedule>
        </job>
    </group>
</config>
```

## Utilizar o CLI 

### Descreva o uso dos comandos bin/magento no ciclo de desenvolvimento. 

**Quais comandos estão disponíveis?**
Uma lista completa de comandos pode ser encontrada executando `bin/magento`. 
Todos os comandos podem ser executados no diretório raiz da instalação Magento através da linha de comando, prefixada com `bin/magento`.

**Como os comandos são usados no ciclo de desenvolvimento?**
Os comandos da CLI fornecem um ponto de entrada seguro para a realização de operações que podem ser inseguras para serem executadas no painel de administração do Magento. O acesso SSH deve ser uma maneira segura de verificar o status de autorização de um usuário.
A ativação de módulos e a execução de scripts de instalação devem ser feitos usando a linha de comando durante o desenvolvimento. Geralmente, é mais rápido alternar o cache das seções ou liberá-lo durante o desenvolvimento usando a linha de comando em comparação com a interface administrativa.


## Descrever como as extensões são instaladas e configuradas
**Como você instalaria e verificaria uma extensão a partir da solicitação do cliente?**
Extenções podem ser instaladas diretamente no diretório `app/code` ou através do composer (no diretório `vendor`). Após os arquivos estarem no local correto, o módulo é habilitado `bin/mangento module:enable nome_modulo` e, então, é usado o `bin/magento setup:upgrade` para registrá-lo na aplicação.

