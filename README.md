# Action para Windows

Captura e transcrição de reuniões com Vox local, histórico por projeto e um
servidor MCP para o agente da sua IDE publicar insights no painel.
O código-fonte do app é privado; este repositório distribui os binários assinados
para Windows e a ponte do Copilot.

**[Baixar Action v1.6.1](https://github.com/AllanSantos-DV/action-releases/releases/tag/v1.6.1)** ·
[Guia visual](https://allansantos-dv.github.io/copilot-marketplace/p/action-bridge/) ·
[Última versão](https://github.com/AllanSantos-DV/action-releases/releases/latest)

## Instalar

1. Baixe **Action-v1.6.1-win64.zip** na release.
2. Extraia todo o conteúdo numa pasta permanente, como
   `%LOCALAPPDATA%\Programs\Action`. Mantenha `_internal/` ao lado dos executáveis.
3. Execute `Action.exe` para a interface do app, ou configure o MCP abaixo
   para o fluxo de reunião com o agente e o painel `/live`.

Requisitos: Windows 10/11, dispositivo de áudio, Vox Engine local para a
transcrição padrão e o host de agente escolhido. A memória compartilhada
permite correlações por projeto. O pacote do Action não requer Python ou Node;
seu host e a ponte Copilot têm os próprios requisitos.

`objects-v1.6.1.zip`, o manifesto e as assinaturas são arquivos do atualizador.
Não use o ZIP de objetos como instalador.

## Reunião pela bandeja

O Action instalado inicia na bandeja com o Windows. Clique no ícone e escolha
**Nova reunião…** para selecionar o projeto, o microfone e a saída de áudio
usada pelo Teams. Clique **Iniciar** quando estiver pronto. Em Configurações,
é possível desligar o início com Windows.

## Claude Code, Codex e Cursor

No PowerShell, dentro da pasta extraída, execute o setup do host que vai usar:

| Host | Comando |
| --- | --- |
| Claude Code | `.\ClaudeCode\setup.ps1` |
| Codex app/CLI (CLI 0.158.0+) | `.\Codex\setup.ps1` |
| Extensão Codex na IDE | `.\Codex\setup.ps1 -Mode Skills` |
| Cursor Desktop/CLI | `.\Cursor\setup.ps1` |

Escolha um modo Codex por perfil. Abra uma nova sessão na pasta do projeto,
confira `/mcp` e, no Codex, revise/confie os hooks Action em `/hooks`.
Os guias completos estão em `ClaudeCode/README.md`, `Codex/README.md` e
`Cursor/README.md` no ZIP. No Claude Desktop, use a aba Code no modo Local.

```text
# Claude Code ou Cursor
/action-join revisão Alpha
/action-record Alpha guardar áudio

# Codex
$action-join revisão Alpha
$action-record Alpha guardar áudio
```

O agente consulta reuniões/projetos pelo nome, pode entrar depois do início e
recebe o histórico anterior. Vários agentes podem acompanhar a mesma reunião;
não há código nem ID para copiar manualmente. O agente pertence à sua sessão,
e o LLM/provedor é o que você configurou na IDE.

Para sair durante o acompanhamento, envie **“Saia da reunião agora. Não encerre
a gravação.”** Sair mantém a captura. Peça explicitamente para encerrar a
reunião quando quiser finalizar a gravação. Os hooks Codex/Cursor mantêm a continuidade
na mesma sessão enquanto habilitados e com o host aberto; respeitam saída,
interrupção e falhas persistentes.

## Tela, MCP e memória

Uma gravação iniciada na tela aparece no MCP, permite entrada tardia do agente e
mostra seus insights na própria interface. Painel web, MCP e tela compartilham
reuniões ativas e histórico salvo. Escolha/crie o projeto e selecione os canais
na tela; o agente usa o mesmo projeto pela memória compartilhada.

Se a memória ficar indisponível, o Action salva localmente e retenta a entrega.
`index_queue_status` e `index_pending` permitem acompanhar esses envios.
A indexação e os embeddings continuam no daemon de memória.

## Copilot

A ponte **action-bridge 1.3.0** fica em `plugin/` e na
[vitrine Copilot](https://allansantos-dv.github.io/copilot-marketplace/p/action-bridge/).
Ela pode instalar o Action quando ausente e expõe suas tools ao agente.

```text
copilot plugin marketplace add AllanSantos-DV/copilot-marketplace
copilot plugin install action-bridge@copilot-marketplace
```

## Áudio, nuvem e atualização

Na tela ou no painel `/live`, escolha o fone usado pelo Teams para captar o
som remoto. O modo Automático segue o padrão global do Windows. Se o Teams
usa outro dispositivo, selecione seu nome; durante a gravação, use **Trocar
saída** sem perder os trechos anteriores. Pode haver uma lacuna curta na troca.
Pelo MCP, `audio_devices.loopback_outputs` lista nomes para
`output_device_name` em `recording_prepare` e `recording_start`.


- Vox local é o padrão. Guardar `mic.wav` e `system.wav` separados é opcional,
  disponível no painel e pelo pedido de guardar áudio ao agente.
- Whisper Groq exige seleção e consentimento explícitos: envia o áudio à nuvem.
  Chaves ficam em variáveis de ambiente. Transcrições enviadas à IDE seguem o
  provedor do seu agente.
- O app oferece atualização por manifesto/pacotes assinados e fallback do ZIP
  completo. Ao migrar da 1.4.0/1.5.0, feche sessões IDE conectadas. Se o atualizador
  antigo continuar na versão anterior, extraia manualmente o novo ZIP na pasta
  de instalação; a 1.5.1 também corrige o encerramento na bandeja durante o update.
- Depois de atualizar, os setups podem atualizar a integração com backups:
  Claude usa `-UpdateMcp -UpdateCommands`; Codex e Cursor usam
  `-UpdateMcp -UpdateIntegration`.

## Validação

Veja as notas da release para os ensaios realizados e limites conhecidos.
A troca de loopback Realtek → Fuxi-H3 foi exercitada nesta máquina sem encerrar
a captura; a conferência de som e transcrição do Teams na tela é do responsável.
Cursor foi validado na CLI nativa: três insights, retomada após silêncio e saída
sem parar a gravação. Os testes cobrem Qt → MCP e recuperação da conexão com o
daemon real. A revisão visual de Cursor Desktop, Codex app/extensão e Claude
Desktop Code/Local será executada pelo responsável. Qualidade e benchmarks do
reconhecimento de voz pertencem ao Vox Engine.
