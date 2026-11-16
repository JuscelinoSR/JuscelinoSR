# PRD — JSR Church Live AI
**Versão:** 1.0 (planejamento)  
**Data:** 2026-11-16  
**Estado:** Documentação inicial; implementação NÃO iniciada  
**Responsável pelo produto:** JuscelinoSR

## 1. Problema e visão
A igreja está sem equipe técnica suficiente e precisa manter as transmissões de cultos com operação mínima, confiabilidade, possibilidade de controle à distância e continuidade quando o responsável não estiver presente.

**Visão:** plataforma local-first para automatizar, monitorar e permitir supervisão remota da transmissão, com acompanhamento PTZ do pregador e fallback seguro.

**Princípio:** confiabilidade da live antes de recursos avançados de IA. Nunca prometer 100% de autonomia; definir operador/contato de contingência para energia, som, rede e falhas físicas.

## 2. Metas e indicadores
- Diminuir intervenções manuais durante o culto.
- Garantir que haja sempre uma cena segura caso o rastreamento falhe.
- Medir: tempo de live disponível; interrupções por culto; incidentes de áudio/vídeo; tempo de recuperação; intervenções humanas; percentual do tempo em enquadramento aceitável.
- Estabelecer metas numéricas só após levantamento da linha de base na Fase 0.

## 3. Contexto conhecido e hipóteses
- Câmera Hikvision PTZ DS-2DE4425IW-DE, operada via navegador; **modelo/firmware/interfaces devem ser confirmados no equipamento**.
- OBS Studio usado na transmissão; verificar versão, cenas, fontes, áudio, WebSocket, hardware e conexão.
- Desejo: rastrear pregador automaticamente no altar; permitir intervenção por celular/notebook de fora da igreja; eventualmente um LLM para interpretar comandos de alto nível.
- Ainda NÃO confirmado: Smart Tracking nativo, ONVIF/ISAPI PTZ habilitado, hardware disponível, rede, tipo de áudio, plataforma da live, horários, permissões de TI, políticas da igreja e autorização de gravação.
- Não registrar IP interno, senhas, tokens, chaves de transmissão, imagens de membros ou credenciais neste repositório.

## 4. Personas e jornadas
1. **Responsável remoto:** visualiza estado da live e alertas; assume câmera/OBS temporariamente; devolve controle ao modo automático.
2. **Voluntário local eventual:** liga equipamentos e aplica procedimentos simples de contingência.
3. **Administrador:** configura políticas, horários, limites PTZ, permissões e revisa logs.

Jornada principal: pré-checagem -> preparo da cena segura -> início supervisionado -> transmissão -> rastreamento opcional -> alarmes / intervenção -> encerramento confirmado -> relatório.

## 5. Escopo por prioridade
### MVP (transmissão confiável, sem IA)
- Inventário e diagrama do ambiente real.
- Backup das configurações atuais; cenas OBS com fallback (plano aberto/púlpito fixo).
- Checklist operacional pré-culto e pós-culto.
- Detecção de OBS em execução, streaming ativo/inativo, presença de vídeo/áudio e silêncio prolongado (limites ajustáveis), conectividade.
- Painel local somente leitura do estado + alertas; logs sem dados sensíveis.
- Plano de recuperação e teste em janela fora do culto.

### V1 (controle PTZ local e remoto)
- Confirmar API do dispositivo; testar conexão e comandos PTZ em laboratório.
- Presets de púlpito, altar, louvor e plano geral; movimento, zoom, stop.
- Controles de acesso; painel remoto via VPN privada, sem exposição direta da câmera/OBS na internet.
- Prioridade de controle: parada de emergência > manual local > manual remoto autorizado > automático.
- Timeout e stop de comandos; retorno a preset seguro quando houver falha.

### V2 (visão computacional)
- Captura do vídeo pela rede local; detectar pessoas; usuário seleciona alvo inicial sem identificação biométrica.
- Rastreador com ID temporário, zona de altar, dead zone, suavização, velocidade máxima, limites de pan/tilt/zoom.
- Política de perda de alvo: parar PTZ, ampliar/voltar para preset seguro; **não seguir automaticamente outra pessoa**.
- Primeiro simular em vídeos autorizados; depois ensaiar com câmera fora da transmissão; ativação em live somente após aprovação.

### V3 (diretor IA opcional)
- LLM interpreta intenções de alto nível (plano médio, presets, limites e preferências de enquadramento).
- O modelo NÃO comanda motores por quadro; o controlador determinístico local aplica limites e confirma autorização.
- Pode operar com modelo local ou API, após avaliação de custos, privacidade e latência.
- Não enviar vídeo, áudio ou dados pessoais para nuvem sem autorização explícita.

## 6. Fora do escopo inicial
- Treinar LLM do zero.
- Identificação facial/biométrica de fiéis.
- Trocas automáticas irrestritas de cenas durante culto.
- Publicar câmera/ONVIF/OBS diretamente na internet.
- Alterar roteador, firmware ou credenciais sem aprovação.
- Comprar GPU, câmeras ou assinaturas antes dos testes de capacidade.

## 7. Arquitetura proposta (lógica, não decisão fechada)
- **Agente local** no computador da igreja: status OBS, health checks e comandos.
- **OBS adapter**: WebSocket quando disponível, com autenticação.
- **PTZ adapter**: ONVIF/ISAPI após teste de compatibilidade, sem credenciais embutidas.
- **Tracking engine**: Python + visão computacional local.
- **Controller**: regras e estados finitos para controlar PTZ, prioridades e fallback.
- **Painel web**: status, presets, iniciar/parar rastreamento, logs, parada de emergência.
- **Acesso remoto**: VPN privada com autenticação e autorização por papel.
- **Diretor LLM** (opcional): transforma instruções em configurações validadas.

## 8. Requisitos funcionais
- RF01: mostrar estado do OBS, stream, áudio, vídeo, conectividade.
- RF02: registrar alertas e recuperar evento com horário e severidade.
- RF03: permitir operador desativar automação e parar PTZ imediatamente.
- RF04: executar presets manuais e obter feedback de comando.
- RF05: garantir controle exclusivo da câmera; não permitir concorrência entre operadores/IA.
- RF06: rastrear somente alvo selecionado na zona configurada.
- RF07: ajustar movimento/zoom suavemente com limites.
- RF08: retomar cena/preset seguro em perda de alvo ou falha.
- RF09: acesso remoto autenticado sem portas públicas de câmera.
- RF10: produzir relatório pós-culto e histórico de incidentes.
- RF11: programação de início/fim somente após aprovação e estratégia de contingência.
- RF12: guardar configuração versionada sem segredos.

## 9. Requisitos não funcionais e segurança
- Local-first; IA automática continua sem internet externa se LAN/PC operacionais.
- Fail-safe: movimento cessa em exceção, timeout ou perda de controle.
- Preservar áudio da mesa/microfones e caminho de vídeo atual até homologação.
- Testar desempenho sem comprometer codificação OBS; limitar carga CPU/GPU.
- Segredos em gerenciador local/variáveis de ambiente; .gitignore adequado.
- Logs minimizados; sem frames de pessoas por padrão; consentimento e políticas da igreja para gravação.
- Acesso remoto com VPN, autenticação forte e princípio do menor privilégio.
- Código e documentação isolados dos projetos Saneago/BRK.
- Toda mudança em produção exige plano de rollback e aprovação.

## 10. Plano sequencial e critérios de saída
| Fase | Entrega | Critério para avançar |
|---|---|---|
| 0 — Descoberta | Inventário do PC, OBS, câmeras, áudio, rede, horários, permissões; diagrama e riscos | Informações verificadas e autorização registrada |
| 1 — Estabilidade | Backup, cenas seguras, checklist, monitoramento e teste de recuperação | Live de ensaio estável e rollback ensaiado |
| 2 — PTZ manual | Adapter PTZ, presets, stop, painel local | Comandos testados fora do culto, limites confirmados |
| 3 — Remoto | VPN, autenticação, perfis, override manual | Teste de acesso/revogação/falha de conexão aprovado |
| 4 — Rastreamento offline | Detector e tracker em vídeo autorizado com métricas | Sem troca indevida de alvo; enquadramento e desempenho aceitos |
| 5 — Rastreamento ao vivo | Integração PTZ com fallback e supervisão | Ensaios repetidos, stop e perda de alvo validados |
| 6 — Diretor IA (opcional) | Comandos naturais com validação estruturada | Nenhum comando contorna limites; custo e privacidade aprovados |
| 7 — Operação assistida | Runbook, agenda, responsáveis, alertas e revisão pós-culto | Operação repetível e responsável de contingência definido |

**Gate obrigatório:** não começar a próxima fase enquanto critérios da anterior estiverem pendentes. Sem produção antes dos ensaios.

## 11. Governança anti-duplicidade e anti-conflitos
1. Este PRD é a **fonte da verdade** de escopo, prioridades e dependências.
2. Uma tarefa por funcionalidade. Antes de criar código, consultar PRD + inventário de módulos + issues existentes.
3. Cada módulo tem responsabilidade única: OBS, PTZ, tracking, controle, painel, IA.
4. Não criar segundo controlador PTZ ou segundo serviço de monitoramento para resolver o mesmo problema; refatorar módulo existente.
5. Tarefas devem ter ID, objetivo, arquivos impactados, dependências, testes, critério de pronto e rollback.
6. Alterações de escopo entram em seção de decisões (ADR) e changelog antes do desenvolvimento.
7. Preferir branches `feat/ID-descricao`, PR com revisão, e commits pequenos; nunca substituir versão estável sem backup.
8. A cada fase: documentação -> protótipo isolado -> teste -> aprovação -> merge -> release.
9. Registrar dúvidas como **A DEFINIR**, nunca como fatos confirmados.
10. Quando retomarmos, verificar este PRD e status da fase corrente ANTES de gerar qualquer código.

## 12. Estrutura de repositório prevista (não criada ainda)
```text
jsr-church-live-ai/
  README.md
  docs/PRD.md
  docs/INVENTARIO.md
  docs/DECISOES.md
  docs/RUNBOOK.md
  docs/FASES.md
  src/obs/
  src/ptz/
  src/tracking/
  src/controller/
  src/dashboard/
  tests/
  .env.example
  .gitignore
```
Criar somente quando a fase justificar; **não gerar módulos vazios**.

## 13. Riscos, mitigação e dependências
- Tracking segue pessoa errada -> seleção manual inicial; zona; stop; fallback.
- OBS perde quadros por GPU/CPU -> profiling e limite de recursos antes de uso real.
- Queda de internet -> serviço local continua; alerta e recuperação supervisionada.
- Queda de energia/PC -> UPS/retomada somente após avaliação.
- Controle remoto comprometido -> VPN, papéis, logs e revogação.
- Áudio ausente -> detecção de silêncio e plano manual local.
- Firmware/API não suportados -> protótipo de controle e eventual alternativa, sem compra prematura.

## 14. Custos e decisões abertas
- Estimativa de partida: R$ 0 em licenças open source, **se** PC/câmera/rede existentes forem suficientes.
- Eventuais gastos com GPU, conectividade, UPS, áudio, VPN, API de IA ou mão de obra só após inventário e orçamento.
- Confirmar: PC (CPU/RAM/GPU), câmera/firmware, protocolos, OBS, streaming, rede e áudio.
- Confirmar: agenda de cultos, voluntário de contingência, autorização de uso e retenção de vídeo.
- Confirmar: plataforma de transmissão, requisitos de agenda e prioridade de câmera versus confiabilidade da live.

## 15. Próxima ação autorizada
**Somente Fase 0 — Descoberta e inventário.** Não criar scripts, instalar componentes, alterar a câmera/OBS, expor serviços ou habilitar auto-tracking antes do levantamento, autorização e revisão deste PRD.

## 16. Histórico
- 2026-11-16 — v1.0: PRD inicial baseado nas necessidades relatadas na conversa.