# triggers-function
em outras linguagens é o PROCEDURE
```SQL
CREATE FUNCTION funcionarios_gatilho() RETURNS TRIGGER AS $funcionarios_gatilho$
  BEGIN
  IF NEW.nome IS NULL THEN
    RAISE EXCEPTION 'o nome do funcionario não pode ser nulo';
  END IF;
  --
IF NEW.salario IS NULL THEN
    RAISE EXCEPTION ' % não pode ter salario nulo', NEW.nome;
  END IF;
  --
IF NEW.salario < 0 THEN
    RAISE EXCEPTION '% não pode pagar para trabalhar', NEW.nome;
  END IF;
  NEW.ultima_data:= 'NOW';
  NEW.ultimo_usuario:= current_user;
  RETURN NEW;
  END;
  $funcionarios_gatilho$
  LANGUAGE plpgsql;
    
  --criando o gatilho trigger
  CREATE TRIGGER funcionarios_gatilho BEFORE
  INSERT OR UPDATE ON  funcionarios
  FOR EACH ROW EXECUTE PROCEDURE funcionarios_gatilho();
  
  CREATE TABLE IF NOT EXISTS funcionario2(
    nome VARCHAR(20) NOT NULL,
    salario NUMERIC(10,2)
    );
   ```
    ```SQL
CREATE TABLE IF NOT EXISTS funcionario_auditoria(
  operacao VARCHAR(20) NOT NULL,
  usuario VARCHAR(20) NOT NULL,
  DATA TIMESTAMP NOT NULL,
  nome_atual VARCHAR(40) NOT NULL,
  salario_atual NUMERIC(10,2),
  nome_antigo VARCHAR(40),
  salario_antigo NUMERIC(10,2)
  );

--TRIGGER FUNCTION QUE VAI ALIMENTAR A TABELA funcionario_auditoria

CREATE OR REPLACE FUNCTION processa_func_audit() RETURNS
TRIGGER AS $log$
BEGIN
  IF (TG_OP = 'DELETE') THEN
  INSERT INTO funcionario_auditoria SELECT 'EXCLUSAO', USER, NOW(), NEW.*, OLD.*;
  RETURN OLD;
  ELSIF (TG_OP = 'UPDATE') THEN
  INSERT INTO funcionario_auditoria SELECT 'ATUALIZACAO', USER, NOW(), NEW.*, OLD.*;
  RETURN NEW;
   ELSIF (TG_OP = 'INSERT') THEN
  INSERT INTO funcionario_auditoria SELECT 'INSERCAO', USER, NOW(), NEW.*;
  RETURN NEW;
  END IF;
  RETURN NULL;
END;
$log$ LANGUAGE plpgsql;

CREATE TRIGGER func_audit
AFTER INSERT OR UPDATE OR DELETE
ON funcionario2
FOR EACH ROW EXECUTE PROCEDURE processa_func_audit();
```
