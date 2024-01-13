# Personalizando_Acessos_Automatizando_acoes_MySQL
Neste desafio você irá criar visões para os seguintes cenários <br/>

Número de empregados por departamento e localidade <br/>

Lista de departamentos e seus gerentes <br/>

Projetos com maior número de empregados (ex: por ordenação desc) <br/>

Lista de projetos, departamentos e gerentes <br/>

Quais empregados possuem dependentes e se são gerentes <br/>
## Criação de Views
- Número de Empregados por Departamento e Localidade<br/>
CREATE VIEW view_empregados_por_departamento AS<br/>
SELECT department_id, location_id, COUNT(*) AS numero_empregados<br/>
FROM employees<br/>
GROUP BY department_id, location_id;<br/>
- Lista de Departamentos e Seus Gerentes<br/>
CREATE VIEW view_departamentos_gerentes AS<br/>
SELECT department_id, department_name, manager_id<br/>
FROM departments;<br/>
- Projetos com Maior Número de Empregados<br/>
CREATE VIEW view_projetos_maior_numero_empregados AS<br/>
SELECT project_id, COUNT(*) AS numero_empregados<br/>
FROM project_employee<br/>
GROUP BY project_id<br/>
ORDER BY numero_empregados DESC;<br/>
- Lista de Projetos, Departamentos e Gerentes<br/>
  CREATE VIEW view_projetos_departamentos_gerentes AS<br/>
SELECT p.project_id, p.project_name, d.department_id, d.department_name, d.manager_id<br/>
FROM projects p<br/>
JOIN departments d ON p.department_id = d.department_id;<br/>
- Empregados com Dependentes e Se São Gerentes<br/>
CREATE VIEW view_empregados_dependentes_gerentes AS<br/>
SELECT e.employee_id, e.first_name, e.last_name, d.dependent_name, e.job_id, e.manager_id<br/>
FROM employees e<br/>
LEFT JOIN dependents d ON e.employee_id = d.employee_id;<br/>
- Permissões de Acesso<br/>
-- Criando um usuário gerente<br/>
CREATE USER 'gerente'@'localhost' IDENTIFIED BY 'senha_gerente';<br/>
GRANT SELECT ON company.view_empregados_por_departamento TO 'gerente'@'localhost';<br/>
GRANT SELECT ON company.view_departamentos_gerentes TO 'gerente'@'localhost';<br/>

## Criando trigger

- Remoção de Usuários e Manutenção de Informações<br/>
DELIMITER //<br/>

CREATE TRIGGER before_delete_usuario<br/>
BEFORE DELETE ON usuarios<br/>
FOR EACH ROW<br/>
BEGIN<br/>
    INSERT INTO log_usuarios_removidos (usuario_id, nome, data_remocao)<br/>
    VALUES (OLD.usuario_id, OLD.nome, NOW());<br/>
END //<br/>

DELIMITER ;<br/>
- Inserção de Novos Colaboradores e Atualização do Salário Base
DELIMITER //<br/>

CREATE TRIGGER before_update_colaboradores<br/>
BEFORE UPDATE ON colaboradores<br/>
FOR EACH ROW<br/>
BEGIN<br/>
    IF NEW.salario_base > OLD.salario_base THEN<br/>
        UPDATE historico_aumento_salario<br/>
        SET data_aumento = NOW(),<br/>
            salario_anterior = OLD.salario_base,<br/>
            salario_atual = NEW.salario_base<br/>
        WHERE colaborador_id = NEW.colaborador_id;<br/>
    END IF;<br/>
END //<br/>

DELIMITER ;<br/>


