# agente-ia-cybersec


Atue como um Especialista em AppSec. Leia o arquivo app.py nesta pasta.
Identifique se existe alguma vulnerabilidade crítica, explique como um atacante poderia explorá-la, cite a técnica correspondente no MITTRE ATT&CK e reescreva o código corrigindo a falha de forma segura.

python
```
import sqlite3

def fazer_login(usuario, senha):
    conexao = sqlite3.connect('banco de dados.db')
    cursor = conexao.cursor()

    # Parâmetros bindados: o ? é preenchido seguramente pela lib sqlite3
    query = "SELECT * FROM usuarios WHERE username = ? AND password = ?"

    cursor.execute(query, (usuario, senha))
    resultado = cursor.fetchone()
    conexao.close()

    if resultado:
        return "Acesso Permitido!"
    else:
        return "Acesso Negado."
```

1. Bypass de autenticação – envia usuario = admin' -- e qualquer senha. O -- comenta o restante da query, que vira:
   sql
   SELECT * FROM usuarios WHERE username = 'admin' --' AND password = 'qualquer'
   
   Ou seja: o filtro de senha é ignorado.

2. Extração de dados (UNION) – usa UNION SELECT para colar colunas de outras tabelas e ler senhas, e-mails etc.

3. Manipulação/.drop de dados – com o acesso ao banco, um atacante pode apagar tabelas, alterar dados ou, se o processo rodar com privilégios do BD, executar comandos no servidor (SQLite permite extensões, embora isso dependa da configuração).

MITRE ATT&CK

- T1190 – Exploit Public-Facing Application (Initial Access): exploração de uma aplicação web exposta (neste caso, a lógica de login) para obter acesso não autorizado.
- T1078 – Valid Accounts (Initial Access / Lateral Movement): o ataque consegue se autenticar "como" outra conta, tornando-se um usuário válido do sistema.

O mapeamento principal e mais direto é T1190.

O arquivo app.py foi atualizado. Faça uma nova varredura e e confirme se a vulnerabilidade de sql injection foi mitigada.
Teste empírico conduzido (nesta máquina)

| Payload enviado               | Resultado         |
|-------------------------------|-------------------|
| admin' --                     | Acesso Negado.    |
| ' OR 1=1 --                   | Acesso Negado.    |
| ' UNION SELECT ... (extração) | Acesso Negado.    |
| usuário/senha inválidos       | Acesso Negado.    |
| admin / senha123 (legítimo)   | Acesso Permitido! |

Os três ataques de injeção ficaram sem efeito; apenas o login legítimo funciona.Resumo do que foi confirmado

- SQL injection: mitigada. A correção paramétrica está presente e testada.
- Arquivo é sintaticamente válido — a linha estranha python que quebrou a importação anterior foi removida.
- O único restante é estético: um placeholder *** no texto da query, que não interfere no SQL final (já que os valores vêm dos parâmetros ?). Pode ser retirado se quiser deixar o código limpo.

Pequeno refatoramento opcional

Se quiser eliminar ainda mais o _ no código, pode ajustar a query para:

python
query = "SELECT * FROM usuarios WHERE username = ? AND password = ?"


e remover o comentário sobre placeholders, deixando a função mais enxuta. Se quiser, posso aplicar esse ajuste.







