# 🔓 Liberar SAP_ALL com DEBUG Ativo

## Objetivo

Conceder permissão `SAP_ALL` (acesso total ao sistema) **apenas durante sessões de DEBUG**, revogando automaticamente após o término.

## ⚠️ Avisos Críticos de Segurança

- 🔴 **CRÍTICO:** SAP_ALL concede acesso total - use com extrema cautela
- 🔴 **LOG:** Todas as ações são registradas em audit trail
- 🔴 **REVOGAÇÃO:** Sempre revogue após o debug terminar
- 🔴 **PRODUÇÃO:** NUNCA use em produção sem aprovação
- 🔴 **CONFORMIDADE:** Viole políticas de segurança corporativas se mal usado

---

## Conceito

Você pode conceder `SAP_ALL` para um usuário, mas com **breakpoints estratégicos** que ativam **apenas durante debug**:

1. **Breakpoint em Authority-Check** - Intercepta verificações de autoridade
2. **Breakpoint em auth_check_tcode** - Intercepta verificações de transação

Assim, o usuário pode acessar o que precisa **sem liberar permanentemente** o ambiente.

---

## Método 1: Authority-Check Breakpoint

### Conceito

O comando `AUTHORITY-CHECK` no ABAP valida se o usuário tem permissão para uma ação.

Colocando um breakpoint aqui, você pode:
- ✅ Ver exatamente qual autoridade está sendo verificada
- ✅ Modificar o resultado em tempo real (sem alterar código)
- ✅ Liberar acesso apenas para debug

### Passo 1: Acessar o Debugger

```
Abrir Debug Mode:
1. Qualquer programa ABAP
2. Pressione: Ctrl + F1
   OU
3. Menu: Program → Utilities → Debugging
```

### Passo 2: Localizar AUTHORITY-CHECK

No código ABAP, procure por:

```ABAP
AUTHORITY-CHECK OBJECT 'ACTIV_REP'
  ID 'ACTVT' FIELD '03'    " 03 = Change/Alter
  ID 'PROG' FIELD 'ZPROG'.
```

### Passo 3: Colocar Breakpoint

Ao chegar nesta linha durante debug:

```
Menu Debug → Breakpoint → Set Breakpoint at Line
(Ou clique no número da linha)
```

Quando o debug para no breakpoint:

```
Stack Display mostra:
  Line: AUTHORITY-CHECK
  sy-subrc (antes): Vazio
```

### Passo 4: Modificar sy-subrc

Depois que o breakpoint para:

```
1. Abrir "Variables" ou "Watch"
2. Procurar: sy-subrc
3. Mudar valor: 0 (sucesso, autorização concedida)
4. Continuar execução: F8 ou Step
```

**Resultado:** Autoridade concedida apenas nesta execução!

### Passo 5: Continuar Debugger

```
Pressione F8 (Go) para continuar
Ou F5 (Step) para próxima linha
```

---

## Método 2: auth_check_tcode Breakpoint

### Conceito

Se o usuário tenta acessar uma transação, o SAP chama internamente `auth_check_tcode`.

Breakpoint aqui permite:
- ✅ Interceptar tentativa de acessar transação
- ✅ Forçar acesso mesmo sem permissão
- ✅ Validar qual transação está sendo acessada

### Passo 1: Ativar Debug Universalmente

```
Transação: /h (Debug Mode Global)
ou
Transação: /SAPL/DEBUG_ACTIVATE
```

### Passo 2: Localizar Função auth_check_tcode

Na transação que deseja acessar:

```
Transação: SE80 (Object Navigator)
1. Buscar pela função: auth_check_tcode
   (Geralmente em módulos de função do kernel)
```

Ou use:

```ABAP
" Localizar no código
SEARCH 'auth_check_tcode'.
```

### Passo 3: Colocar Breakpoint

```
Menu: Debug → Breakpoint → Breakpoint at Function Module
Entrar: auth_check_tcode
```

### Passo 4: Tentar Acessar Transação

```
Digite transação (ex: /nSU01)
Debug para no breakpoint de auth_check_tcode
```

### Passo 5: Modificar Retorno

Dentro do breakpoint:

```ABAP
" Variáveis importantes:
" exception_not_raised = .TRUE.  (sucesso)
" exception_not_raised = .FALSE. (falha)

1. Abrir Variables
2. Mudar exception_not_raised = .TRUE.
3. Continuar (F8)
```

**Resultado:** Acesso à transação concedido!

---

## Passo a Passo Completo: Liberar SAP_ALL com Debug

### Cenário Real

```
Você precisa acessar transação SU01 (User Admin) 
para testes, mas não tem permissão.
```

### Execução

#### **Passo 1: Ativar Debug Global**

```
Transação: /h
(Ativa debug mode para todas as ações)
```

#### **Passo 2: Tentar Acessar Transação Restrita**

```
Digite na barra: /nSU01
SAP tenta verificar autoridade
DEBUG PARA automaticamente
```

#### **Passo 3: Debug para em auth_check_tcode**

```
Call Stack mostra:
  → auth_check_tcode
  → sy-subrc = 4 (acesso negado)
```

#### **Passo 4: Modificar em Tempo Real**

```
1. Clicar em "Variables" (Locals)
2. Procurar: sy-subrc
3. Clicar duplo e mudar para: 0
4. Pressionar Enter
```

#### **Passo 5: Continuar**

```
Pressionar F8 (Continue/Go)
Transação SU01 abre normalmente!
```

#### **Passo 6: Usar Transação Livremente**

```
Agora você pode:
- Consultar usuários
- Criar usuários
- Modificar roles
- ETC
(Tudo em modo debug, rastreável)
```

#### **Passo 7: Desativar Debug**

```
Transação: /h (novamente)
DEBUG OFF
Acesso revogado automaticamente
```

---

## Técnica Avançada: AUTHORITY-CHECK no Código

### Se você é desenvolvedor e edita código:

```ABAP
" Localizar no programa:
AUTHORITY-CHECK OBJECT 'S_DEVELOP'
  ID 'DEVCLASS' FIELD devclass
  ID 'ACTIVITY' FIELD '01'.

IF sy-subrc <> 0.
  MESSAGE E101 WITH 'Sem permissão'.
ENDIF.
```

### Colocar breakpoint exatamente aqui:

```
1. Clicar número da linha
2. Menu: Debug → Breakpoint → Set
3. Executar programa (F8)
4. Quando parar no breakpoint:
```

### Modificar inline:

```
5. Abrir "Variables"
6. Duplo-clique em sy-subrc
7. Mudar para 0
8. Continue (F8)
```

**Pronto!** Autoridade concedida apenas nesta execução.

---

## Modo Debug: Atalhos Úteis

| Tecla | Função |
|-------|--------|
| `Ctrl + F1` | Iniciar/Parar Debug |
| `/h` | Debug Global (qualquer tela) |
| `/H` | Debug em próxima ação |
| `F5` | Step Into (próxima linha) |
| `F6` | Step Over (próxima função) |
| `F8` | Continue (Go) |
| `F9` | Execute até cursor |
| `Shift + F8` | Sair do Debug |

---

## Variáveis Chave no Debug

### sy-subrc (System Return Code)

```ABAP
" 0 = Sucesso (acesso concedido)
sy-subrc = 0.    ✅ Autoridade OK

" 4 = Acesso negado
sy-subrc = 4.    ❌ Sem permissão

" 8 = Erro fatal
sy-subrc = 8.    ❌ Erro de sistema
```

### exception_not_raised

```ABAP
" .TRUE. = Autoridade concedida
exception_not_raised = .TRUE.    ✅

" .FALSE. = Acesso negado
exception_not_raised = .FALSE.   ❌
```

---

## Exemplo Visual: Tela de Debug

```
┌─────────────────────────────────────────────────────┐
│ Debugger - Step Through                             │
├─────────────────────────────────────────────────────┤
│                                                     │
│  AUTHORITY-CHECK OBJECT 'S_DEVELOP'    ← BREAKPOINT│
│    ID 'DEVCLASS' FIELD devclass                     │
│    ID 'ACTIVITY' FIELD '01'.                        │
│                                                     │
├─────────────────────────────────────────────────────┤
│ Variables:                                          │
│  sy-subrc            [4]        ← Mude para: 0     │
│  sy-msgty            [W]                            │
│  exception_not_raised [.FALSE.] ← Mude para: TRUE  │
│                                                     │
├─────────────────────────────────────────────────────┤
│ F5:Step F6:Over F8:Go Shift+F8:Exit                │
└─────────────────────────────────────────────────────┘
```

---

## ✅ Checklist de Segurança

**ANTES de usar SAP_ALL com Debug:**

- [ ] Tenho autorização do administrador?
- [ ] Estou em QAS/DEV (não PROD)?
- [ ] Documentei por que preciso de debug?
- [ ] Entendo as implicações de segurança?
- [ ] Tenho supervisor acompanhando?
- [ ] Registro de todas as ações será mantido?

**DEPOIS de usar:**

- [ ] Desativei o debug (/h)?
- [ ] Revoquei SAP_ALL se foi concedida?
- [ ] Documentei as ações realizadas?
- [ ] Alertei o time de segurança?
- [ ] Fecharei a sessão imediatamente?

---

## 🚨 O Que NÃO Fazer

### ❌ Uso Indevido

```
NÃO:
- Deixar debug ativo sem monitoramento
- Usar SAP_ALL em produção sem motivo
- Esconder alterações realizadas em debug
- Modificar dados sem documentação
- Compartilhar credenciais com debug ativo
```

### ✅ Uso Correto

```
SIM:
- Debug com supervisor
- Registre tudo em log
- Use ambiente de testes
- Revogue acesso imediatamente
- Documente cada ação
```

---

## Referências

### Documentação SAP

- [Debugger ABAP - SAP Help](https://help.sap.com/doc/abapdocu_latest/en-US/)
- [Authorization Checks - SAP Security](https://help.sap.com/doc/abapdocu_latest/en-US/)
- [Debug Mode - SAP Guide](https://help.sap.com/doc/abapdocu_latest/en-US/)

### Comandos Relacionados

```ABAP
" Verificar autoridades do usuário
SUBMIT SU53 AND RETURN.

" Listar falhas de autorização
SUBMIT SU56 AND RETURN.

" Rastrear autoridades (audit trail)
SUBMIT SM20 AND RETURN.
```

---

## Conclusão

Usar breakpoints em `AUTHORITY-CHECK` e `auth_check_tcode` é uma técnica **poderosa** para debugging, mas requer:

✅ **Responsabilidade**  
✅ **Documentação**  
✅ **Supervisão**  
✅ **Revogação Imediata**  

Use apenas em ambientes controlados com aprovação adequada.

---

**Baseado em:** SAP Authorization & Debug System  
**Versão:** PT-BR  
**Última atualização:** 26/05/2026  
**⚠️ CONFIDENCIAL - Uso Restrito**
