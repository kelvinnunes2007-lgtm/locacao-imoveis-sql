```markdown
# Schema SQLite para Locação de Imóveis — versão com validações

Este arquivo contém um script SQLite com validações adicionais para um sistema de gestão de locação de imóveis:
- PRAGMA para habilitar foreign keys
- NOT NULL e UNIQUE onde faz sentido (ex.: cpf_cnpj)
- CHECK para enumerações (status, prioridade)
- colunas created_at / updated_at com triggers para manter updated_at atualizado
- índices em chaves estrangeiras
- regras de ON DELETE / ON UPDATE pensadas para manter integridade

Observação: execute `PRAGMA foreign_keys = ON;` antes de usar o banco para que as FKs sejam aplicadas.

```sql
-- Ativar enforcement de foreign keys
PRAGMA foreign_keys = ON;

-- TABELAS PRINCIPAIS
CREATE TABLE proprietario (
  id_proprietario INTEGER PRIMARY KEY AUTOINCREMENT,
  nome TEXT NOT NULL,
  cpf_cnpj TEXT NOT NULL UNIQUE,
  telefone TEXT,
  email TEXT UNIQUE,
  chave_pix TEXT,
  created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
  updated_at DATETIME DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE imovel (
  id_imovel INTEGER PRIMARY KEY AUTOINCREMENT,
  bairro TEXT NOT NULL,
  cidade TEXT NOT NULL,
  estado TEXT NOT NULL,
  cep TEXT,
  endereco TEXT NOT NULL,
  id_proprietario INTEGER,
  created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
  updated_at DATETIME DEFAULT CURRENT_TIMESTAMP,
  FOREIGN KEY (id_proprietario) REFERENCES proprietario(id_proprietario)
    ON DELETE SET NULL ON UPDATE CASCADE
);

CREATE TABLE inquilino (
  id_inquilino INTEGER PRIMARY KEY AUTOINCREMENT,
  nome TEXT NOT NULL,
  cpf_cnpj TEXT NOT NULL UNIQUE,
  telefone TEXT,
  email TEXT UNIQUE,
  informacoes_trabalho TEXT,
  created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
  updated_at DATETIME DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE unidade (
  id_unidade INTEGER PRIMARY KEY AUTOINCREMENT,
  numero_unidade TEXT NOT NULL,
  status TEXT NOT NULL DEFAULT 'Disponível' CHECK(status IN ('Disponível','Ocupado','Manutenção','Reservado')),
  aluguel_sugerido REAL NOT NULL DEFAULT 0.0,
  id_imovel INTEGER NOT NULL,
  created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
  updated_at DATETIME DEFAULT CURRENT_TIMESTAMP,
  FOREIGN KEY (id_imovel) REFERENCES imovel(id_imovel)
    ON DELETE CASCADE ON UPDATE CASCADE
);

CREATE TABLE contrato_locacao (
  id_contrato INTEGER PRIMARY KEY AUTOINCREMENT,
  data_inicio DATE NOT NULL,
  data_fim DATE NOT NULL,
  aluguel_mensal REAL NOT NULL,
  valor_caucao REAL DEFAULT 0.0,
  tipo_garantia TEXT,
  status TEXT NOT NULL DEFAULT 'Ativo' CHECK(status IN ('Ativo','Encerrado','Renovado','Cancelado')),
  id_unidade INTEGER NOT NULL,
  id_inquilino INTEGER NOT NULL,
  id_proprietario INTEGER NOT NULL,
  created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
  updated_at DATETIME DEFAULT CURRENT_TIMESTAMP,
  FOREIGN KEY (id_unidade) REFERENCES unidade(id_unidade) ON DELETE RESTRICT ON UPDATE CASCADE,
  FOREIGN KEY (id_inquilino) REFERENCES inquilino(id_inquilino) ON DELETE RESTRICT ON UPDATE CASCADE,
  FOREIGN KEY (id_proprietario) REFERENCES proprietario(id_proprietario) ON DELETE RESTRICT ON UPDATE CASCADE
);

CREATE TABLE pagamento (
  id_pagamento INTEGER PRIMARY KEY AUTOINCREMENT,
  valor REAL NOT NULL,
  data_vencimento DATE NOT NULL,
  data_pagamento DATE,
  forma_pagamento TEXT,
  status TEXT NOT NULL DEFAULT 'Pendente' CHECK(status IN ('Pendente','Pago','Atrasado','Cancelado')),
  observacoes TEXT,
  id_contrato INTEGER NOT NULL,
  created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
  updated_at DATETIME DEFAULT CURRENT_TIMESTAMP,
  FOREIGN KEY (id_contrato) REFERENCES contrato_locacao(id_contrato) ON DELETE CASCADE ON UPDATE CASCADE
);

CREATE TABLE documento (
  id_documento INTEGER PRIMARY KEY AUTOINCREMENT,
  entidade_relacionada TEXT,
  id_entidade_relacionada INTEGER,
  titulo TEXT NOT NULL,
  arquivo_link TEXT,
  data_upload DATE DEFAULT (date('now')),
  id_contrato INTEGER,
  created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
  updated_at DATETIME DEFAULT CURRENT_TIMESTAMP,
  FOREIGN KEY (id_contrato) REFERENCES contrato_locacao(id_contrato) ON DELETE CASCADE ON UPDATE CASCADE
);

CREATE TABLE cobranca_extra (
  id_cobranca INTEGER PRIMARY KEY AUTOINCREMENT,
  descricao TEXT NOT NULL,
  valor REAL NOT NULL DEFAULT 0.0,
  data_vencimento DATE,
  status TEXT NOT NULL DEFAULT 'Aberto' CHECK(status IN ('Aberto','Pago','Cancelado')),
  id_contrato INTEGER NOT NULL,
  created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
  updated_at DATETIME DEFAULT CURRENT_TIMESTAMP,
  FOREIGN KEY (id_contrato) REFERENCES contrato_locacao(id_contrato) ON DELETE CASCADE ON UPDATE CASCADE
);

CREATE TABLE vistoria (
  id_vistoria INTEGER PRIMARY KEY AUTOINCREMENT,
  data DATE NOT NULL,
  entrada_saida TEXT NOT NULL CHECK(entrada_saida IN ('Entrada','Saída')),
  responsavel TEXT,
  relatorio TEXT,
  observacoes_estado TEXT,
  id_contrato INTEGER NOT NULL,
  created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
  updated_at DATETIME DEFAULT CURRENT_TIMESTAMP,
  FOREIGN KEY (id_contrato) REFERENCES contrato_locacao(id_contrato) ON DELETE CASCADE ON UPDATE CASCADE
);

CREATE TABLE solicitacao_manutencao (
  id_manutencao INTEGER PRIMARY KEY AUTOINCREMENT,
  data_abertura DATE NOT NULL,
  prioridade TEXT NOT NULL DEFAULT 'Média' CHECK(prioridade IN ('Baixa','Média','Alta')),
  status TEXT NOT NULL DEFAULT 'Aberto' CHECK(status IN ('Aberto','Em Progresso','Concluído','Cancelado')),
  descricao TEXT NOT NULL,
  custo REAL DEFAULT 0.0,
  prestador_servico TEXT,
  id_unidade INTEGER,
  id_inquilino INTEGER,
  created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
  updated_at DATETIME DEFAULT CURRENT_TIMESTAMP,
  FOREIGN KEY (id_unidade) REFERENCES unidade(id_unidade) ON DELETE SET NULL ON UPDATE CASCADE,
  FOREIGN KEY (id_inquilino) REFERENCES inquilino(id_inquilino) ON DELETE SET NULL ON UPDATE CASCADE
);

-- INDICES PARA PERFORMANCE (especialmente em colunas de FK)
CREATE INDEX idx_imovel_id_proprietario ON imovel(id_proprietario);
CREATE INDEX idx_unidade_id_imovel ON unidade(id_imovel);
CREATE INDEX idx_contrato_id_unidade ON contrato_locacao(id_unidade);
CREATE INDEX idx_contrato_id_inquilino ON contrato_locacao(id_inquilino);
CREATE INDEX idx_pagamento_id_contrato ON pagamento(id_contrato);
CREATE INDEX idx_manutencao_id_unidade ON solicitacao_manutencao(id_unidade);
CREATE INDEX idx_manutencao_id_inquilino ON solicitacao_manutencao(id_inquilino);

-- TRIGGERS PARA ATUALIZAR updated_at AUTOMATICAMENTE
CREATE TRIGGER trg_proprietario_updated_at
AFTER UPDATE ON proprietario
FOR EACH ROW
BEGIN
  UPDATE proprietario SET updated_at = CURRENT_TIMESTAMP WHERE id_proprietario = OLD.id_proprietario;
END;

CREATE TRIGGER trg_inquilino_updated_at
AFTER UPDATE ON inquilino
FOR EACH ROW
BEGIN
  UPDATE inquilino SET updated_at = CURRENT_TIMESTAMP WHERE id_inquilino = OLD.id_inquilino;
END;

CREATE TRIGGER trg_imovel_updated_at
AFTER UPDATE ON imovel
FOR EACH ROW
BEGIN
  UPDATE imovel SET updated_at = CURRENT_TIMESTAMP WHERE id_imovel = OLD.id_imovel;
END;

CREATE TRIGGER trg_unidade_updated_at
AFTER UPDATE ON unidade
FOR EACH ROW
BEGIN
  UPDATE unidade SET updated_at = CURRENT_TIMESTAMP WHERE id_unidade = OLD.id_unidade;
END;

CREATE TRIGGER trg_contrato_updated_at
AFTER UPDATE ON contrato_locacao
FOR EACH ROW
BEGIN
  UPDATE contrato_locacao SET updated_at = CURRENT_TIMESTAMP WHERE id_contrato = OLD.id_contrato;
END;

CREATE TRIGGER trg_pagamento_updated_at
AFTER UPDATE ON pagamento
FOR EACH ROW
BEGIN
  UPDATE pagamento SET updated_at = CURRENT_TIMESTAMP WHERE id_pagamento = OLD.id_pagamento;
END;

CREATE TRIGGER trg_manutencao_updated_at
AFTER UPDATE ON solicitacao_manutencao
FOR EACH ROW
BEGIN
  UPDATE solicitacao_manutencao SET updated_at = CURRENT_TIMESTAMP WHERE id_manutencao = OLD.id_manutencao;
END;

-- DADOS DE EXEMPLO (compatíveis com constraints)
INSERT INTO proprietario (nome, cpf_cnpj, telefone, email, chave_pix)
VALUES
('João da Silva', '12345678900', '11999999999', 'joao@gmail.com', 'joao@pix.com'),
('Maria Souza', '98765432100', '11988888888', 'maria@gmail.com', 'maria@pix.com');

INSERT INTO imovel (bairro, cidade, estado, cep, endereco, id_proprietario)
VALUES
('Centro', 'São Paulo', 'SP', '01001-000', 'Rua A, 100', 1),
('Jardins', 'São Paulo', 'SP', '01401-010', 'Rua B, 200', 2);

INSERT INTO unidade (numero_unidade, status, aluguel_sugerido, id_imovel)
VALUES
('101', 'Disponível', 2500.00, 1),
('102', 'Ocupado', 2200.00, 1),
('201', 'Disponível', 3000.00, 2);

INSERT INTO inquilino (nome, cpf_cnpj, telefone, email, informacoes_trabalho)
VALUES
('Carlos Pereira', '45678912300', '11977777777', 'carlos@gmail.com', 'Analista de TI'),
('Ana Lima', '78912345600', '11966666666', 'ana@gmail.com', 'Designer');

INSERT INTO contrato_locacao (data_inicio, data_fim, aluguel_mensal, valor_caucao, tipo_garantia, status, id_unidade, id_inquilino, id_proprietario)
VALUES
('2024-01-10', '2025-01-10', 2500.00, 2500.00, 'Caução', 'Ativo', 1, 1, 1),
('2024-03-05', '2025-03-05', 2200.00, 2200.00, 'Fiador', 'Ativo', 2, 2, 1);

INSERT INTO pagamento (valor, data_vencimento, data_pagamento, forma_pagamento, status, observacoes, id_contrato)
VALUES
(2500.00, '2024-02-10', '2024-02-09', 'PIX', 'Pago', 'Pagamento antecipado', 1),
(2200.00, '2024-04-05', NULL, 'Boleto', 'Pendente', '', 2);

INSERT INTO cobranca_extra (descricao, valor, data_vencimento, status, id_contrato)
VALUES
('Troca de lâmpada', 50.00, '2024-02-15', 'Pago', 1),
('Multa por atraso', 100.00, '2024-04-10', 'Aberto', 2);

INSERT INTO vistoria (data, entrada_saida, responsavel, relatorio, observacoes_estado, id_contrato)
VALUES
('2024-01-10', 'Entrada', 'João Fiscal', 'Imóvel em bom estado', '', 1);

INSERT INTO solicitacao_manutencao (data_abertura, prioridade, status, descricao, custo, prestador_servico, id_unidade, id_inquilino)
VALUES
('2024-02-20', 'Alta', 'Aberto', 'Vazamento na pia', 150.00, 'Hidráulica SP', 1, 1);

-- EXEMPLOS DE CONSULTAS ÚTEIS
-- 1. Listar unidades disponíveis por imóvel
SELECT i.endereco, u.numero_unidade, u.status
FROM imovel i
JOIN unidade u ON u.id_imovel = i.id_imovel
WHERE u.status = 'Disponível';

-- 2. Contratos com dados completos
SELECT c.id_contrato, inq.nome AS inquilino, u.numero_unidade, c.aluguel_mensal
FROM contrato_locacao c
JOIN inquilino inq ON inq.id_inquilino = c.id_inquilino
JOIN unidade u ON u.id_unidade = c.id_unidade;

-- 3. Pagamentos pendentes
SELECT * FROM pagamento WHERE status = 'Pendente' ORDER BY data_vencimento ASC;

-- 4. Unidades com maior aluguel sugerido (top 2)
SELECT * FROM unidade ORDER BY aluguel_sugerido DESC LIMIT 2;

-- 5. Solicitações de manutenção abertas
SELECT s.*, inq.nome FROM solicitacao_manutencao s
LEFT JOIN inquilino inq ON inq.id_inquilino = s.id_inquilino
WHERE s.status = 'Aberto';

-- EXEMPLOS DE ATUALIZAÇÃO
-- Atualizar status de pagamento (usa data atual do SQLite)
UPDATE pagamento SET status = 'Pago', data_pagamento = date('now') WHERE id_pagamento = 2;

-- Atualizar aluguel sugerido
UPDATE unidade SET aluguel_sugerido = 2800.00 WHERE id_unidade = 1;

-- Marcar manutenção como concluída
UPDATE solicitacao_manutencao SET status = 'Concluído' WHERE id_manutencao = 1;

-- EXEMPLOS DE DELETES (cuidado: algumas FKs usam CASCADE)
-- Excluir cobranças pagas
DELETE FROM cobranca_extra WHERE status = 'Pago';

-- Excluir manutenção concluída
DELETE FROM solicitacao_manutencao WHERE status = 'Concluído';

-- Excluir contratos antigos
DELETE FROM contrato_locacao WHERE data_fim < '2023-01-01';
