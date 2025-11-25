  DER---Biblioteca

Descrição:
Fiz um pequeno plano de como funcionaria um sistema de banco de dados de uma biblioteca, com entidades como Livro, Autor, Usuário, Empréstimo e Histórico.

Tecnologias / Ferramentas usadas
- Modelagem: draw.io (diagrama DER)
- Banco: MySQL (MySQL Workbench / MySQL Benchmark usado para testes)

Links
- Diagrama online (draw.io): https://app.diagrams.net/#G1Bmrflc3T4gdzqLsXaOxSLSuttCNFOhSY#%7B%22pageId%22%3A%22PTh_fs_BfjOOXzCjD59z%22%7D  
- Arquivo (.drawio / export): https://drive.google.com/file/d/1Bmrflc3T4gdzqLsXaOxSLSuttCNFOhSY/view?usp=sharing

O que tem neste repositório
- Diagrama DER (arquivo .drawio ou imagem exportada)
- Sugestões de tabelas e chaves primárias/estrangeiras
- Exemplos de queries (se desejar, posso adicionar um arquivo .sql com CREATE TABLE e inserts de exemplo)

Como visualizar
1. Abra o link do draw.io ou faça o download do arquivo .drawio do Google Drive.  
2. No draw.io, escolha "Abrir do dispositivo" para carregar o arquivo localmente.
script SQL: -- Banco de dados: biblioteca_db
CREATE DATABASE IF NOT EXISTS biblioteca_db
  DEFAULT CHARACTER SET = utf8mb4
  DEFAULT COLLATE = utf8mb4_unicode_ci;
USE biblioteca_db;

-- Autores
CREATE TABLE IF NOT EXISTS autor (
  autor_id BIGINT AUTO_INCREMENT PRIMARY KEY,
  nome VARCHAR(200) NOT NULL
) ENGINE=InnoDB;

-- Categorias / gêneros
CREATE TABLE IF NOT EXISTS categoria (
  categoria_id BIGINT AUTO_INCREMENT PRIMARY KEY,
  nome VARCHAR(100) NOT NULL
) ENGINE=InnoDB;

-- Livros (metadados)
CREATE TABLE IF NOT EXISTS livro (
  livro_id BIGINT AUTO_INCREMENT PRIMARY KEY,
  titulo VARCHAR(300) NOT NULL,
  isbn VARCHAR(20),
  ano_publicacao YEAR,
  categoria_id BIGINT,
  quantidade_total INT DEFAULT 1,
  criado_em TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  FOREIGN KEY (categoria_id) REFERENCES categoria(categoria_id) ON DELETE SET NULL
) ENGINE=InnoDB;

-- Relação muitos-para-muitos: livro <-> autor
CREATE TABLE IF NOT EXISTS livro_autor (
  livro_id BIGINT NOT NULL,
  autor_id BIGINT NOT NULL,
  PRIMARY KEY (livro_id, autor_id),
  FOREIGN KEY (livro_id) REFERENCES livro(livro_id) ON DELETE CASCADE,
  FOREIGN KEY (autor_id) REFERENCES autor(autor_id) ON DELETE CASCADE
) ENGINE=InnoDB;

-- Cópias físicas (ex.: várias cópias do mesmo livro)
CREATE TABLE IF NOT EXISTS copia (
  copia_id BIGINT AUTO_INCREMENT PRIMARY KEY,
  livro_id BIGINT NOT NULL,
  numero_copia INT NOT NULL DEFAULT 1,
  estado ENUM('disponivel','emprestado','perdido','danificado') DEFAULT 'disponivel',
  FOREIGN KEY (livro_id) REFERENCES livro(livro_id) ON DELETE CASCADE
) ENGINE=InnoDB;

-- Membros / usuários da biblioteca
CREATE TABLE IF NOT EXISTS membro (
  membro_id BIGINT AUTO_INCREMENT PRIMARY KEY,
  nome VARCHAR(200) NOT NULL,
  email VARCHAR(200),
  telefone VARCHAR(30),
  data_cadastro DATE DEFAULT CURDATE()
) ENGINE=InnoDB;

-- Empréstimos (um empréstimo pode ter múltiplas cópias)
CREATE TABLE IF NOT EXISTS emprestimo (
  emprestimo_id BIGINT AUTO_INCREMENT PRIMARY KEY,
  membro_id BIGINT NOT NULL,
  data_emprestimo DATE NOT NULL,
  data_devolucao_prevista DATE NOT NULL,
  data_devolucao_real DATE DEFAULT NULL,
  status ENUM('aberto','fechado','atrasado') DEFAULT 'aberto',
  FOREIGN KEY (membro_id) REFERENCES membro(membro_id) ON DELETE CASCADE
) ENGINE=InnoDB;

-- Itens do empréstimo (vincula cópias)
CREATE TABLE IF NOT EXISTS emprestimo_item (
  emprestimo_id BIGINT NOT NULL,
  copia_id BIGINT NOT NULL,
  PRIMARY KEY (emprestimo_id, copia_id),
  FOREIGN KEY (emprestimo_id) REFERENCES emprestimo(emprestimo_id) ON DELETE CASCADE,
  FOREIGN KEY (copia_id) REFERENCES copia(copia_id) ON DELETE CASCADE
) ENGINE=InnoDB;

-- Índices úteis
CREATE INDEX idx_livro_titulo ON livro(titulo);
CREATE INDEX idx_membro_nome ON membro(nome);

-- Inserts de exemplo
INSERT INTO categoria (nome) VALUES ('Ficção'), ('Ciência'), ('Tecnologia') 
  ON DUPLICATE KEY UPDATE nome=VALUES(nome);

INSERT INTO autor (nome) VALUES ('Machado de Assis'), ('Isaac Asimov'), ('Autor Exemplo')
  ON DUPLICATE KEY UPDATE nome=VALUES(nome);

INSERT INTO livro (titulo, isbn, ano_publicacao, categoria_id, quantidade_total) VALUES
  ('Dom Casmurro', '978-85-0000-000-1', 1899, 1, 2),
  ('Fundação', '978-85-0000-000-2', 1951, 2, 3),
  ('Livro Exemplo', '000-0', 2021, 3, 1)
  ON DUPLICATE KEY UPDATE titulo=VALUES(titulo);

-- vincular autores
INSERT IGNORE INTO livro_autor (livro_id, autor_id) VALUES
  (1, 1),
  (2, 2),
  (3, 3);

-- criar cópias físicas
INSERT INTO copia (livro_id, numero_copia, estado) VALUES
  (1, 1, 'disponivel'),
  (1, 2, 'disponivel'),
  (2, 1, 'disponivel'),
  (2, 2, 'disponivel'),
  (2, 3, 'disponivel'),
  (3, 1, 'disponivel');

-- membros
INSERT INTO membro (nome, email, telefone) VALUES
  ('João Silva', 'joao@example.com', '99999-0000'),
  ('Maria Souza', 'maria@example.com', '98888-1111');

-- exemplo de empréstimo
INSERT INTO emprestimo (membro_id, data_emprestimo, data_devolucao_prevista, status) VALUES
  (1, CURDATE(), DATE_ADD(CURDATE(), INTERVAL 7 DAY), 'aberto');

-- ligar uma cópia ao empréstimo
INSERT INTO emprestimo_item (emprestimo_id, copia_id) VALUES
  (1, 1);

-- atualizar estado da cópia para emprestado
UPDATE copia SET estado='emprestado' WHERE copia_id=1;
