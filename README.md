# 🎄 Cantatas de Natal Boulevard 2026

Protótipo de sistema de inscrição para apresentações de Natal no Boulevard Shopping Vila Velha. O público escolhe data e horário num calendário, preenche um formulário, os dados caem numa planilha do Google e a equipe gerencia tudo num calendário visual.

> ⚠️ **Status: no ar.** Feito com apoio de IA (Claude). 

## Problema

Corais, igrejas, escolas e projetos sociais precisam se inscrever para se apresentar no Natal do shopping. Sem um sistema, é fácil ter horário duplicado, mensagem perdida e ninguém saber o que já foi confirmado. O projeto atende:

- **Público externo:** quer ver datas livres e se inscrever sem criar conta.
- **Equipe do shopping:** quer receber os dados organizados, aprovar e reagendar.

## Funcionalidades

**Para o público**
- Calendário mensal com status por data: Disponível, Aguardando confirmação, Ocupado e Encerrado.
- Ao clicar numa data, um modal mostra horários, status e vagas, com o botão "Inscrever-se".
- Formulário sem login. Obrigatórios: coral, responsável, e-mail, telefone, cidade, integrantes, data e horário. Opcionais: instituição e necessidades técnicas/observações.
- Contadores na página: datas disponíveis, datas ocupadas e corais inscritos (atualizam ao carregar a página e após cada inscrição).
- Bloqueio de horário já reservado, com a mensagem: "Este horário já foi reservado. Por favor, selecione outra data."

**Para a equipe (área protegida por senha)**
- Cards: inscrições, confirmadas, pendentes e datas ocupadas.
- Calendário com cada coral no seu dia, colorido por status, com filtro por status.
- Detalhes do coral e ações: confirmar, concluir, cancelar e reagendar (escolhendo entre horários livres).
- Bloqueio de conflito, com a mensagem: "Já existe uma apresentação nesta data e horário."
- **Aprovação manual:** a inscrição entra como *Pendente* e só ocupa o horário depois que a equipe confirma.

**E-mails automáticos** (enviados pelo Gmail da conta que implantou o app)
- Inscrição recebida (para o coral) e aviso de nova inscrição (para a equipe).
- Apresentação confirmada.
- Alteração de data da apresentação (com nova data e horário).
- Lembretes 15, 7 e 1 dia antes, por gatilho diário, sem reenviar duplicados.

**Configuração (no topo do `Code.gs`):** período de datas, horários, exceções de horário por dia (ex.: 24/12 só às 14h e 17h) e e-mail da equipe.

## Stack

| Camada | Tecnologia |
|---|---|
| Front-end | HTML, CSS e JavaScript puro (fontes Playfair Display e Raleway) |
| Back-end | Google Apps Script (Web App) |
| Banco de dados | Google Sheets |
| E-mails | MailApp (Gmail) |
| Hospedagem | Implantação do Apps Script como "App da Web" |
| Desenvolvimento | Claude (IA) para gerar e iterar o código |

Identidade visual baseada na comunicação "Doce Natal Boulevard" (creme, vermelho, verde e dourado).

## Banco de dados

Uma única aba, `Inscricoes`, sem relações entre tabelas:

| Coluna | Conteúdo |
|---|---|
| ID | Código curto gerado automaticamente |
| Criado em | Data e hora da inscrição |
| Coral, Responsável, E-mail, Telefone, Cidade | Dados do coral |
| Participantes | Quantidade de integrantes |
| Instituição | Opcional |
| Obs./Necessidades técnicas | Opcional |
| Data, Horário | Como texto (evita que a planilha converta o formato) |
| Status | Pendente, Confirmado, Realocado, Concluído ou Cancelado |
| Lembretes | Controle dos lembretes já enviados |

**Regras de acesso (equivalente ao RLS):**
- O público só recebe a disponibilidade (função `estado()`), sem nenhum dado pessoal.
- Funções da equipe (`lista`, `atualiza`) exigem a senha, conferida **no servidor** (propriedade do script `ADMIN_PASS`).
- Confirmado e Realocado ocupam o horário. Pendente, Concluído e Cancelado não ocupam.
- A gravação usa `LockService` para evitar duas inscrições simultâneas no mesmo horário.

> **Sobre o Supabase:** esta versão **não usa** Supabase. Um desenho de banco para uma versão futura (tabelas `corais`, `slots` e `user_roles`, índice único parcial para impedir dois confirmados no mesmo horário, RLS) foi especificado em um prompt, mas **não foi implementado nem validado**.

## Fluxo de uso

1. O coral abre o link e vê o calendário.
2. Clica numa data livre e em "Inscrever-se".
3. Preenche o formulário e envia.
4. Vê a mensagem de que a inscrição aguarda confirmação e recebe um e-mail.
5. A equipe recebe o aviso, entra na área da equipe e confirma, reagenda ou cancela.
6. O coral recebe o e-mail de confirmação (ou de alteração) e os lembretes antes da apresentação.

## Como rodar

Não há servidor local: o app roda dentro do Google Apps Script.

1. Crie uma planilha no Google Sheets e abra **Extensões › Apps Script**.
2. Cole `apps-script/Code.gs` no arquivo `Code.gs` e crie um arquivo HTML chamado `Index` com o conteúdo de `apps-script/Index.html`.
3. No topo do `Code.gs`, troque o e-mail de exemplo pelo da equipe e ajuste datas e horários, se precisar.
4. Em **Configurações do projeto › Propriedades do script**, crie `ADMIN_PASS` com a senha da equipe (nunca coloque a senha no código).
5. Rode a função `setup` uma vez e autorize. Ela cria a aba e o gatilho diário de lembretes.
6. Em **Implantar › Nova implantação › App da Web**, use **Executar como: Eu** e **Quem pode acessar: Qualquer pessoa**.
7. Teste com uma inscrição falsa antes de divulgar. A cada mudança no código, publique uma **Nova versão** da implantação.

Há também um protótipo visual em `prototipo/` (HTML único, dados no `localStorage` do navegador, e-mails simulados, login de demonstração). Ele não grava em nenhum servidor.

## Prints e GIFs sugeridos

- Calendário público com os quatro status.
- Modal de uma data com os horários.
- Formulário e a mensagem de sucesso.
- Calendário da equipe com chips coloridos.
- Modal de detalhes com os botões de ação.
- A planilha `Inscricoes` com linhas de teste.
- GIF: inscrição → confirmação → e-mail recebido.

*(Use dados fictícios ou borre dados reais.)*


## Aprendizados

- Login feito só no navegador não protege nada: a senha precisa ser conferida no servidor.
- Bloquear horário duplicado tem que acontecer na hora de gravar, não só na tela.
- Aprovação manual evita que alguém "segure" uma data e não responda.
- Forçar texto nas colunas de data e hora evita que a planilha reformate os valores.
- O link só abre para o público se o acesso estiver como "Qualquer pessoa", e toda mudança exige uma nova versão da implantação.
- Plataformas com cobrança por atualização podem não compensar para uso interno; Sheets + Apps Script deu mais liberdade para editar.


## Autor e licença

Lucas Santos Nascimento
