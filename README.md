# triggers-function
em outras linguagens é o PROCEDURE

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
  
  
