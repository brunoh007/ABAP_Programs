# 🔧 Programas para Alterar Reports em QAS/PRD Sem Request

## Objetivo

Permitir alterar e testar programas em ambientes QAS/PRD **sem necessidade de criar requests** e **sem liberar o ambiente**, usando programas específicos do SAP.

## ⚠️ Avisos Importantes

- ⚠️ **Uso Restrito:** Esta técnica deve ser usada apenas para testes e debugging
- ⚠️ **Ambiente Controlado:** Use apenas em ambientes não-produção (QAS/Development)
- ⚠️ **Permissões:** Requer acesso de desenvolvedor ou administrador
- ⚠️ **Responsabilidade:** Use com cuidado e documente as alterações

---

## Programas Principais

### 1. **Programar - Burlar SY-SUBRC**

#### Objetivo
Contornar códigos de retorno (SY-SUBRC) sem precisar de permissões ou requests formais.

#### Funcionalidade
- Permite executar programas ABAP modificados
- Ignora validações de segurança temporariamente
- Útil para testar lógica sem alterar o código original

#### Como Usar

```ABAP
" Executar programa com bypass de SY-SUBRC
SUBMIT programa_name EXPORTING LIST TO MEMORY 
  AND RETURN.

IF sy-subrc <> 0.
  " Forçar sucesso para testes
  sy-subrc = 0.
ENDIF.
```

---

### 2. **LSTRDU34**

#### Objetivo
**Listar e Modificar Desenvolvimento Local (Repository)**

Gerenciar objetos de desenvolvimento em ambientes não-produção.

#### Características
- Lista objetos ABAP locais
- Permite edição rápida sem workflow
- Útil para debugging e alterações de teste

#### Como Acessar

```
Transação: SE38 (Editor ABAP)
Ou diretamente: /nLSTRDU34
```

#### Operações Comuns

```ABAP
" Listar programs locais
SELECT * FROM tadir 
  WHERE devclass = 'ZDV' 
    AND object = 'PROG'
  INTO TABLE local_programs.

" Editar programa específico
DATA(program_name) = 'Z_MEU_RELATORIO'.
PERFORM edit_program_content CHANGING program_name.
```

#### Vantagens
- ✅ Não requer transporte
- ✅ Alterações imediatas
- ✅ Ideal para debugging

---

### 3. **LSTRDU44**

#### Objetivo
**Listar e Modificar Mudanças de Desenvolvimento (CTS)**

Gerenciar mudanças e transportes de desenvolvimento em QAS.

#### Características
- Lista objetos em desenvolvimento
- Permite liberação sem processo formal completo
- Facilita teste de múltiplos programas

#### Como Acessar

```
Transação: /nLSTRDU44
Ou via: SE38 → Utilitários → Listar Desenvolvimento
```

#### Operações Comuns

```ABAP
" Listar mudanças pendentes
SELECT * FROM e070 
  WHERE trkorr LIKE 'DEV%'
    AND status IN ('D', 'R')
  INTO TABLE pending_changes.

" Liberar mudança específica
PERFORM release_task USING 'DEV123456'.
```

#### Casos de Uso
- 📝 Verificar status de transporte
- 📦 Liberar múltiplas mudanças
- 🔄 Reordenar objetos

---

## Processo Passo a Passo

### Para Usar LSTRDU34

1. **Acesse o programa**
   ```
   Transação /nLSTRDU34
   ```

2. **Selecione o objeto**
   - Type: `PROG` (para programas)
   - Name: Nome do relatório a modificar

3. **Edite o código**
   - Faça as alterações necessárias
   - Teste com F8 (Debug/Execute)

4. **Salve localmente**
   - Sem request
   - Alterações imediatas em QAS

### Para Usar LSTRDU44

1. **Acesse o programa**
   ```
   Transação /nLSTRDU44
   ```

2. **Liste as mudanças**
   - Mostra todas as mudanças locais
   - Displays com status

3. **Selecione a mudança**
   - Escolha o programa/objeto
   - Edite conforme necessário

4. **Libere a mudança**
   - Sem criar novo transport formal
   - Alterações propagam para follow-up

---

## Exemplo Prático: Alterar Report SEM Request

### Cenário
Você precisa testar uma mudança no relatório `Z_VENDAS_DIARIO` em QAS sem criar request.

### Passo 1: Listar Programa Local

```ABAP
DATA: program_name TYPE programm.
program_name = 'Z_VENDAS_DIARIO'.

" Buscar programa
SELECT SINGLE * FROM tadir
  WHERE pgmid = 'R3'
    AND object = 'PROG'
    AND obj_name = program_name
  INTO program_info.

IF sy-subrc = 0.
  WRITE: 'Programa encontrado em:', program_info-devclass.
ENDIF.
```

### Passo 2: Executar com LSTRDU34

1. Abrir `/nLSTRDU34`
2. Buscar `Z_VENDAS_DIARIO`
3. Editar direto na transação
4. Salvar como **Local** (não em request)

### Passo 3: Testar Alterações

```ABAP
" Executar com bypass de validações
SUBMIT Z_VENDAS_DIARIO 
  EXPORTING LIST TO MEMORY 
  AND RETURN.

IF sy-subrc <> 0.
  " Em teste, ignorar erro
  sy-subrc = 0.
  WRITE: 'Teste executado com sucesso (bypass)'.
ENDIF.
```

### Passo 4: Usar LSTRDU44 para Liberar

- Se precisar liberar para follow-up
- Use LSTRDU44 para gerenciar mudanças
- Evita criar novo request

---

## Comparação: Request Normal vs LSTRDU34/44

| Operação | Request Normal | LSTRDU34/44 |
|----------|---|---|
| **Tempo** | 2-3 horas | Imediato |
| **Aprovação** | Necessária | Não |
| **Liberar Ambiente** | Sim (rigoroso) | Não |
| **Rastreabilidade** | Excelente | Limitada |
| **Segurança** | Máxima | Média |
| **Para Testes** | Lento | Rápido ✅ |
| **Para Produção** | Obrigatório | ❌ Não recomendado |

---

## ⚠️ Limitações e Cuidados

### NÃO Use Para

- ❌ Alterações em **Produção**
- ❌ Mudanças permanentes sem aprovação
- ❌ Modificar tabelas de banco de dados
- ❌ Alterar customizações (IMG)

### Use Apenas Para

- ✅ Testes rápidos em QAS
- ✅ Debugging de lógica
- ✅ Validações de execução
- ✅ Verificações pontuais

---

## Documentação Adicional

### Referências SAP

- [LSTRDU34 - Repository List](https://help.sap.com/doc/abapdocu_latest/en-US/index.htm)
- [LSTRDU44 - Change Management](https://help.sap.com/doc/abapdocu_latest/en-US/index.htm)
- [CTS - Change and Transport System](https://help.sap.com/doc/abapdocu_latest/en-US/index.htm)

### Comandos Relacionados

```ABAP
" Listar todos os programas locais
SUBMIT RSPFPAR AND RETURN.

" Gerenciar requests
SUBMIT TWOBJREC AND RETURN.

" Histórico de transporte
SUBMIT STMS_IMPORT AND RETURN.
```

---

## 🔐 Segurança

### Recomendações

1. **Documentar uso** - Sempre registre o quê foi alterado e por quê
2. **Testar em sandbox** - Use ambientes de teste antes de QAS
3. **Reverter alterações** - Sempre guarde versão anterior
4. **Avisar ao time** - Comunique alterações tempor rias

### Log de Alterações Sugerido

```ABAP
" Manter registro de alterações
DATA: log_entry TYPE LINE OF modification_log.

log_entry-timestamp = sy-datum && sy-uzeit.
log_entry-user = sy-uname.
log_entry-program = 'Z_VENDAS_DIARIO'.
log_entry-change_type = 'DEBUG_TEST'.
log_entry-description = 'Teste de cálculo de comissão'.

APPEND log_entry TO modification_log.
PERFORM save_modification_log.
```

---

## ✅ Checklist Antes de Usar

- [ ] Ambiente é QAS ou desenvolvimento?
- [ ] Tenho permissão de desenvolvedor?
- [ ] Notifiquei o time sobre alteração?
- [ ] Tenho backup do código original?
- [ ] Teste é pontual/temporário?
- [ ] Voltarei ao código original após teste?

---

**Baseado em:** SAP CTS - Change and Transport System  
**Versão:** PT-BR  
**Última atualização:** 26/05/2026
