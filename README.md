# Perity

Controle financeiro pessoal em SwiftUI para iPhone e Mac: quanto você tem em conta, quanto deve no cartão, o que falta pagar no mês e quanto seus investimentos renderam. Versão aberta, feita para quem programa.

Projeto pessoal · iOS 17+ e macOS 14+ · 2026

[Página no portfólio](https://cauecarneiro.github.io/portfolio/perity.html)

App nativo **SwiftUI para iPhone e Mac**, feito para **uso pessoal de quem programa**: você compila, coloca as
**suas** chaves e conecta os **seus** bancos. Seus dados ficam no seu aparelho (e, se quiser, no seu iCloud).
Não existe servidor, conta nem coleta de dados.

> **Leia antes:** este projeto é **não oficial**, sem qualquer ligação com a Pluggy, com bancos ou com a Apple.
> Não é produto para o público geral, não é consultoria financeira e vem **sem garantia**. Veja
> [docs/AVISOS.md](docs/AVISOS.md).

## Telas

Todas com os dados de exemplo que vêm no app (fictícios).

<p>
  <img src="docs/telas/ios-inicio.jpg" width="200" alt="Início no iPhone">
  <img src="docs/telas/ios-investimentos.jpg" width="200" alt="Investimentos no iPhone">
  <img src="docs/telas/ios-fixos.jpg" width="200" alt="Contas fixas no iPhone">
  <img src="docs/telas/ios-saude.jpg" width="200" alt="Saúde financeira no iPhone">
</p>

![Visão geral no Mac](docs/telas/mac-visaoGeral.jpg)

![Investimentos no Mac](docs/telas/mac-investimentos.jpg)

## O que faz

- **Início (o macro):** saldo somado das contas, faturas somadas dos cartões, gastos do mês e fixos a pagar.
  Cada número abre o seu detalhe (o micro). Embaixo: um aviso, o que vence em breve e os últimos lançamentos.
- **Gastos do mês** por categoria (cerca de 56, em 14 áreas) com destaques objetivos: ritmo contra o mês
  passado, o que subiu, quanto cabe por dia. Corrigir uma categoria ensina o app (cria uma regra).
- **Fixos:** a lista das contas do mês com caixinha de pago. O que sai no cartão ou na conta é marcado sozinho
  pelo extrato, com previsão do dia do débito e histórico de 12 meses. Uma varredura sugere o que se repete.
- **Investimentos:** total, quanto rendeu em reais e em %, comparação com o CDI (série pública do Banco Central)
  e o fechamento do mês.
- **Mac:** organizado como um caderno de páginas (barra lateral, tabelas), ícone na barra de menus.
- **Widgets** (iPhone, Mac e tela de bloqueio), **Atalhos/Siri** e **registro na hora de cada compra no Apple Pay**.
- **Privacidade:** modo "ocultar valores" por página e tela borrada na lista de apps recentes.
- **Importar OFX/CSV** para bancos fora das suas conexões, e lançamentos manuais.

## De onde vêm os dados

| Fonte | Como | Frequência |
|---|---|---|
| Bancos, cartões, investimentos | Open Finance via **Meu Pluggy** (uso pessoal; os limites e o plano dependem da Pluggy: confira no site deles) | em ciclos definidos pela Pluggy |
| Compras no Apple Pay | Atalhos → ação "Registrar compra" | na hora |
| Outros bancos | Importar OFX ou CSV | quando importar |
| Dinheiro, cripto, imóvel | Manual | quando quiser |

O app só **lê**. Ele não move dinheiro nem altera nada nos bancos.

## O que o app envia pela rede

Duas conexões vão para a internet em qualquer uso, e uma terceira só existe se você ligar o iCloud:

1. `api.pluggy.ai`: para ler os **seus** dados, usando as **suas** credenciais.
2. `api.bcb.gov.br`: consulta pública do CDI, sem enviar nada seu.
3. **iCloud (CloudKit), só se você ligar `PERITY_ICLOUD=true`:** aí os seus **lançamentos, contas, investimentos e o `itemId` das conexões**
   passam a ser guardados no **seu iCloud privado** (container da sua conta Apple), para sincronizar iPhone e Mac. Ninguém do projeto
   tem acesso a isso, mas é dado financeiro saindo do aparelho (protegido pela sua conta Apple). Com `PERITY_ICLOUD=false`, nada disso acontece.
   O Client ID e o Client Secret **nunca** vão para o iCloud (ficam só no Keychain).

Não há analytics, anúncios nem servidor do projeto. (Dois links em Ajustes, `meu.pluggy.ai` e `dashboard.pluggy.ai`, só abrem
no navegador quando você toca neles.)

## Meu papel

Projeto solo. Eu especifiquei, orquestrei os agentes de IA que ajudaram a escrever o código, revisei e medi o resultado. O repositório tem um único commit, de Cauê Carneiro, com a versão aberta.

## Como foi feito

- **Núcleo separado da interface:** o pacote Swift `Prumo/PrumoCore` guarda modelos, cliente da Pluggy, categorização, conciliação, análises, importador e dados de exemplo. Não tem UI e roda com `swift test`, sem simulador.
- **Telas compartilhadas:** `Prumo/App/Shared` tem as telas SwiftUI, o estado global e os App Intents; `Prumo/App/macOS` tem o ícone na barra de menus.
- **Dados compatíveis com CloudKit:** SwiftData sem `@Attribute(.unique)` e sem relacionamentos (ligações por texto, como `contaID`). Dinheiro é sempre `Int` em centavos, nunca `Double`.
- **Regras que evitam número errado:** transferências, pagamento de fatura e aportes ficam fora de gasto e renda; a conciliação casa compras do Apple Pay com o extrato do banco; a sincronização repetida não duplica lançamentos. Detalhes em [docs/ARQUITETURA.md](docs/ARQUITETURA.md).
- **Projeto gerado:** o `.xcodeproj` nunca é versionado; o `Prumo/project.yml` (XcodeGen) o gera.

## Tecnologias

Swift, SwiftUI, SwiftData, CloudKit, WidgetKit, App Intents, Keychain, XcodeGen, Swift Package Manager, XCTest.

## Como rodar

Você precisa de um Mac com Xcode 26 ou mais novo (o código usa APIs do SDK do iOS 26 e macOS 26, mas roda a partir do iOS 17 e do macOS 14), [XcodeGen](https://github.com/yonaskolb/XcodeGen)
(`brew install xcodegen`) e uma conta Apple (gratuita serve para rodar no seu aparelho; veja os limites abaixo).

```bash
git clone <endereço-do-seu-fork>
cd Perity/Prumo
./Scripts/configurar.sh      # pergunta o SEU prefixo de app e a SUA equipe; cria Configuracao.local (fora do git)
./Scripts/gerar-projeto.sh   # gera Prumo.xcodeproj (nunca é versionado)
open Prumo.xcodeproj         # esquemas: Prumo (iPhone) e PrumoMac (Mac)
```

Depois siga o passo a passo completo em **[docs/COMO-CONECTAR.md](docs/COMO-CONECTAR.md)**: criar as chaves na
Pluggy, ligar o conector MeuPluggy e colar as credenciais **dentro do app** (elas vão para o Keychain).
Para só ver as telas, escolha **"Ver com dados de exemplo"** na abertura: nada da Pluggy precisa ser configurado (os dados de exemplo são fictícios).

Testes do núcleo (sem simulador): `cd Prumo/PrumoCore && swift test`.

### Conta gratuita × conta paga

| | Conta Apple gratuita | Conta de desenvolvedor paga |
|---|---|---|
| Rodar no seu aparelho | sim (o app expira em poucos dias e precisa ser reinstalado) | sim |
| Sincronizar iPhone e Mac pelo iCloud | **não** (deixe `PERITY_ICLOUD=false`) | sim (`PERITY_ICLOUD=true`) |
| Widgets com dados (grupo de app) | pode exigir conta paga, conforme a regra atual da Apple | sim |

Sem iCloud, cada aparelho tem a sua própria base. Isso é suficiente para testar e usar em um aparelho só.

## Onde fica o segredo (e onde nunca fica)

- **Client ID e Client Secret da Pluggy:** só no **Keychain** do aparelho (`Cofre`), digitados dentro do app.
  Nunca em arquivo, nunca no iCloud, nunca no repositório.
- `Configuracao.local` guarda **apenas** o seu prefixo de app, a sua equipe e o liga/desliga do iCloud.
  Está no `.gitignore`.
- Antes de fazer commit no seu fork, confira com `git status` e `git diff --cached`.

Detalhes e como reportar falhas: [SECURITY.md](SECURITY.md).

## Documentos

- [docs/COMO-CONECTAR.md](docs/COMO-CONECTAR.md): passo a passo para usar com os seus dados.
- [docs/PLUGGY.md](docs/PLUGGY.md): o que o app usa da API da Pluggy e como conferir se continua funcionando.
- [docs/ARQUITETURA.md](docs/ARQUITETURA.md): como o app é montado e as regras de negócio.
- [docs/DESIGN.md](docs/DESIGN.md): visual e organização das telas.
- [docs/AVISOS.md](docs/AVISOS.md): avisos legais, privacidade e LGPD.
- [CONTRIBUTING.md](CONTRIBUTING.md): como contribuir sem expor dados.

## Limites (ditos com franqueza)

- O Open Finance **não é tempo real**: atualiza em ciclos definidos pela Pluggy (o intervalo e o plano dependem dela; confira no site deles). O "tempo real" vem do Apple Pay
  (só compras por aproximação/Apple Pay, e alguns emissores não disparam o gatilho do Atalhos).
- Os limites de conexões e o plano do uso pessoal dependem da Pluggy e podem mudar; para mais bancos, importe extratos ou lance manualmente.
- Sincronização por iCloud e automação do Atalhos só podem ser validadas nos seus aparelhos.
- O app abre direto, sem senha própria: o bloqueio do aparelho é a sua proteção.
- O nome "Prumo" no código é o nome antigo do projeto (pasta, pacote, esquemas). O app se chama **Perity**.

## Licença

[MIT](LICENSE). O uso da API da Pluggy segue os termos e planos da própria Pluggy, que não fazem parte desta licença.

---
Feito por Cauê Crillanovick · [portfólio](https://cauecarneiro.github.io/portfolio/) · [LinkedIn](https://www.linkedin.com/in/cau%C3%AA-carneiro)
