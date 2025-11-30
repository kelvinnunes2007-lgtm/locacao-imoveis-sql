CREATE TABLE Proprietario (
 id_proprietario INTEGER PRIMARY KEY AUTOINCREMENT,
  nome TEXT,
  cpf_cnpj TEXT,
  telefone TEXT,
  email TEXT,
  chave_pix TEXT
);

----------TABELA: IMOVEL-----------
CREATE TABLE Imovel(
  id_imovel INTEGER PRIMARY KEY AUTOINCREMENT,
  bairro TEXT,
  cidade TEXT,
  estado TEXT,
  cep TEXT,
  endereco TEXT,
  id_proprietario INTEGER, 
  FOREIGN KEY (id_proprietario) REFERENCES Proprietario(id_proprietario)
);

----------TABELA: UNIDADE------------
CREATE TABLE Unidade (
  id_unidade INTEGER PRIMARY KEY AUTOINCREMENT,
  numero_unidade TEXT,
  status TEXT,
  aluguel_sugerido DECIMAL(10,2),
  id_imovel INTEGER
  FOREIGN KEY (id_imovel) REFERENCES Imovel(id_imovel)
);

----------TABELA: Inquilino----------
CREATE TABLE Inquilino (
  id_inquilino INTEGER PRIMARY KEY AUTOINCREMENT,
  nome TEXT,
  cpf_cnpj TEXT,
  telefone TEXT,
  email TEXT,
  informacoes_trabalho TEXT
);

----------TABELA: CONTRATO LOCACAO---------
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
  FOREIGN KEY (id_proprietario) REFERENCES Proprietario(id_proprietario),
);

---------TABELA: PAGAMENTO--------
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

--------TABLE: DOCUMENTO-------
CREATE TABLE Documento (
  id_documento INTEGER PRIMARY KEY AUTOINCREMENT
  entidade_relacionada TEXT,
  id_entidade_relacionada INTEGER,
  titulo TEXT,
  arquivo_link TEXT,
  data_upload DATE, 
  id_contrato INTEGER,
  FOREIGN KEY (id_contrato) REFERENCES Contrato_locacao(id_contrato)
);

-------TABELA: COBRANCA EXTRA-------
CREATE TABLE Cobranca_extra (
  id_cobranca INTEGER PRIMARY KEY AUTOINCREMENT,
  descricao TEXT,
  valor DECIMAL(10,2),
  data_vencimento DATE,
  status TEXT,
  id_contrato INTEGER,
  FOREIGN KEY (id_contrato) REFERENCES Contrato_Locacao(id_contrato)
);

-------TABELA: VISTORIA---------
CREATE TABLE Vistoria (
  id_vistoria INTEGER PRIMARY KEY AUTOINCREMENT,
  data DATE,
  entrada_saida TEXT,
  responsavel TEXT,
  relatorio TEXT,
  observacoes_estado TEXT,
  id_contrato INTEGER,
  FOREIGN KEY (id_contrato) REFERENCES Contrato_Locacao(id_contrato)
);

------TABELA: SOLICITACAO DE MANUTENCAO---------
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
