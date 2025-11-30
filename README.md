CREATE TABLE Proprietario (
  id_proprietario INTEGER PRIMARY KEY AUTOINCREMENT,
  nome TEXT,
  cpf_cnpj TEXT,
  telefone TEXT,
  email TEXT,
  chave_pix TEXT
);

CREATE TABLE Imovel (
  id_imovel INTEGER PRIMARY KEY AUTOINCREMENT,
  bairro TEXT,
  cidade TEXT,
  estado TEXT,
  cep TEXT,
  endereco TEXT,
  id_proprietario INTEGER,
  FOREIGN KEY (id_proprietario) REFERENCES Proprietario(id_proprietario)
);

CREATE TABLE Unidade (
  id_unidade INTEGER PRIMARY KEY AUTOINCREMENT,
  numero_unidade TEXT,
  status TEXT,
  aluguel_sugerido DECIMAL(10,2),
  id_imovel INTEGER,
  FOREIGN KEY (id_imovel) REFERENCES Imovel(id_imovel)
);

CREATE TABLE Inquilino (
  id_inquilino INTEGER PRIMARY KEY AUTOINCREMENT,
  nome TEXT,
  cpf_cnpj TEXT,
  telefone TEXT,
  email TEXT,
  informacoes_trabalho TEXT
);

CREATE TABLE Contrato_locacao (
  id_contrato INTEGER PRIMARY KEY AUTOINCREMENT,
  data_inicio DATE,
  data_fim DATE,
  aluguel_mensal DECIMAL(10,2),
  valor_caucao DECIMAL(10,2),
  tipo_garantia TEXT,
  status TEXT,
  id_unidade INTEGER,
  id_inquilino INTEGER,
  id_proprietario INTEGER,
  FOREIGN KEY (id_unidade) REFERENCES Unidade(id_unidade),
  FOREIGN KEY (id_inquilino) REFERENCES Inquilino(id_inquilino),
  FOREIGN KEY (id_proprietario) REFERENCES Proprietario(id_proprietario)
);

CREATE TABLE Pagamento (
  id_pagamento INTEGER PRIMARY KEY AUTOINCREMENT,
  valor DECIMAL(10,2),
  data_vencimento DATE,
  data_pagamento DATE,
  forma_pagamento TEXT,
  status TEXT,
  observacoes TEXT,
  id_contrato INTEGER,
  FOREIGN KEY (id_contrato) REFERENCES Contrato_locacao(id_contrato)
);

CREATE TABLE Documento (
  id_documento INTEGER PRIMARY KEY AUTOINCREMENT,
  entidade_relacionada TEXT,
  id_entidade_relacionada INTEGER,
  titulo TEXT,
  arquivo_link TEXT,
  data_upload DATE,
  id_contrato INTEGER,
  FOREIGN KEY (id_contrato) REFERENCES Contrato_locacao(id_contrato)
);

CREATE TABLE Cobranca_extra (
  id_cobranca INTEGER PRIMARY KEY AUTOINCREMENT,
  descricao TEXT,
  valor DECIMAL(10,2),
  data_vencimento DATE,
  status TEXT,
  id_contrato INTEGER,
  FOREIGN KEY (id_contrato) REFERENCES Contrato_locacao(id_contrato)
);

CREATE TABLE Vistoria (
  id_vistoria INTEGER PRIMARY KEY AUTOINCREMENT,
  data DATE,
  entrada_saida TEXT,
  responsavel TEXT,
  relatorio TEXT,
  observacoes_estado TEXT,
  id_contrato INTEGER,
  FOREIGN KEY (id_contrato) REFERENCES Contrato_locacao(id_contrato)
);

CREATE TABLE Solicitacao_manutencao (
  id_manutencao INTEGER PRIMARY KEY AUTOINCREMENT,
  data_abertura DATE,
  prioridade TEXT,
  status TEXT,
  descricao TEXT,
  custo DECIMAL(10,2),
  prestador_servico TEXT,
  id_unidade INTEGER,
  id_inquilino INTEGER,
  FOREIGN KEY (id_unidade) REFERENCES Unidade(id_unidade),
  FOREIGN KEY (id_inquilino) REFERENCES Inquilino(id_inquilino)
);
-- PROPRIETARIOS
INSERT INTO Proprietario (nome, cpf_cnpj, telefone, email, chave_pix)
VALUES
('João da Silva', '12345678900', '11999999999', 'joao@gmail.com', 'joao@pix'),
('Maria Souza', '98765432100', '11988888888', 'maria@gmail.com', 'maria@pix');

-- IMOVEIS
INSERT INTO Imovel (bairro, cidade, estado, cep, endereco, id_proprietario)
VALUES
('Centro', 'São Paulo', 'SP', '01001-000', 'Rua A, 100', 1),
('Jardins', 'São Paulo', 'SP', '01401-010', 'Rua B, 200', 2);

-- UNIDADES
INSERT INTO Unidade (numero_unidade, status, aluguel_sugerido, id_imovel)
VALUES
('101', 'Disponível', 2500.00, 1),
('102', 'Ocupado', 2200.00, 1),
('201', 'Disponível', 3000.00, 2);

-- INQUILINOS
INSERT INTO Inquilino (nome, cpf_cnpj, telefone, email, informacoes_trabalho)
VALUES
('Carlos Pereira', '45678912300', '11977777777', 'carlos@gmail.com', 'Analista de TI'),
('Ana Lima', '78912345600', '11966666666', 'ana@gmail.com', 'Designer');

-- CONTRATOS
INSERT INTO Contrato_Locacao (data_inicio, data_fim, aluguel_mensal, valor_caucao, tipo_garantia, status, id_unidade, id_inquilino, id_proprietario)
VALUES
('2024-01-10', '2025-01-10', 2500.00, 2500.00, 'Caução', 'Ativo', 1, 1, 1),
('2024-03-05', '2025-03-05', 2200.00, 2200.00, 'Fiador', 'Ativo', 2, 2, 1);

-- PAGAMENTOS
INSERT INTO Pagamento (valor, data_vencimento, data_pagamento, forma_pagamento, status, observacoes, id_contrato)
VALUES
(2500.00, '2024-02-10', '2024-02-09', 'PIX', 'Pago', 'Pagamento antecipado', 1),
(2200.00, '2024-04-05', NULL, 'Boleto', 'Pendente', '', 2);

-- COBRANÇAS EXTRAS
INSERT INTO Cobranca_Extra (descricao, valor, data_vencimento, status, id_contrato)
VALUES
('Troca de lâmpada', 50.00, '2024-02-15', 'Pago', 1),
('Multa por atraso', 100.00, '2024-04-10', 'Aberto', 2);

-- VISTORIAS
INSERT INTO Vistoria (data, entrada_saida, responsavel, relatorio, observacoes_estado, id_contrato)
VALUES
('2024-01-10', 'Entrada', 'João Fiscal', 'Imóvel em bom estado', '', 1);

-- SOLICITAÇÕES DE MANUTENÇÃO
INSERT INTO Solicitacao_Manutencao (data_abertura, prioridade, status, descricao, custo, prestador_servico, id_unidade, id_inquilino)
VALUES
('2024-02-20', 'Alta', 'Aberto', 'Vazamento na pia', 150.00, 'Hidráulica SP', 1, 1);
-- 1. Listar imóveis e suas unidades disponíveis
SELECT i.endereco, u.numero_unidade, u.status
FROM Imovel i
JOIN Unidade u ON u.id_imovel = i.id_imovel
WHERE u.status = 'Disponível';

-- 2. Buscar contratos com dados completos
SELECT c.id_contrato, inq.nome AS inquilino, u.numero_unidade, c.aluguel_mensal
FROM Contrato_Locacao c
JOIN Inquilino inq ON inq.id_inquilino = c.id_inquilino
JOIN Unidade u ON u.id_unidade = c.id_unidade;

-- 3. Pagamentos pendentes
SELECT * FROM Pagamento
WHERE status = 'Pendente'
ORDER BY data_vencimento ASC;

-- 4. Unidades com maior valor sugerido (TOP 2)
SELECT * FROM Unidade
ORDER BY aluguel_sugerido DESC
LIMIT 2;

-- 5. Solicitações de manutenção ABERTAS
SELECT s.*, inq.nome
FROM Solicitacao_Manutencao s
JOIN Inquilino inq ON inq.id_inquilino = s.id_inquilino
WHERE s.status = 'Aberto';
-- 1. Atualizar status de pagamento
UPDATE Pagamento
SET status = 'Pago', data_pagamento = CURDATE()
WHERE id_pagamento = 2;

-- 2. Atualizar valor de aluguel sugerido
UPDATE Unidade
SET aluguel_sugerido = 2800.00
WHERE id_unidade = 1;

-- 3. Atualizar status de solicitação de manutenção
UPDATE Solicitacao_Manutencao
SET status = 'Concluído'
WHERE id_manutencao = 1;
-- 1. Excluir cobrança extra quitada
DELETE FROM Cobranca_Extra
WHERE status = 'Pago';

-- 2. Excluir manutenção concluída
DELETE FROM Solicitacao_Manutencao
WHERE status = 'Concluído';

-- 3. Excluir contratos antigos (exemplo)
DELETE FROM Contrato_Locacao
WHERE data_fim < '2023-01-01';
