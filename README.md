import pygame
import random
import math
import sys

# Inicialização do Pygame
pygame.init()

# Configurações da Tela
LARGURA, ALTURA = 1200, 750
tela = pygame.display.set_mode((LARGURA, ALTURA))
pygame.display.set_caption("Mini MOBA - Versão Celular com Game Over")
relogio = pygame.time.Clock()

# Cores
COR_FUNDO = (15, 15, 20)
COR_PAREDE = (35, 45, 80)
COR_PLAYER = (50, 220, 90)  # Verde
COR_ALIADO = (240, 200, 20)  # Amarelo
COR_INIMIGO = (230, 50, 50)  # Vermelho
COR_TIRO = (255, 255, 255)

# Cores Especiais para Habilidades
COR_MINA = (255, 165, 0)
COR_DARDO = (200, 50, 255)
COR_ONDA = (255, 50, 50)

# Configurações do Botão Virtual da Ultimate
BOTAO_X, BOTAO_Y = 100, 650
BOTAO_RAIO = 45
COR_BOTAO = (255, 165, 0)

# Configurações dos Botões da Tela de Game Over
REINICIAR_RECT = pygame.Rect(LARGURA // 2 - 160, ALTURA // 2 + 20, 140, 50)
FECHAR_RECT = pygame.Rect(LARGURA // 2 + 20, ALTURA // 2 + 20, 140, 50)

# Configurações do Labirinto Expandido
TAM_BLOCO = 50
MATRIZ_MAPA = [
    [1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1],
    [1, 0, 0, 0, 0, 0, 1, 0, 0, 0, 0, 0, 0, 0, 0, 0, 1, 0, 0, 0, 0, 0, 0, 1],
    [1, 0, 1, 1, 1, 0, 1, 0, 1, 1, 1, 1, 1, 1, 0, 0, 1, 0, 1, 1, 1, 1, 0, 1],
    [1, 0, 1, 0, 0, 0, 0, 0, 1, 0, 0, 0, 0, 1, 0, 0, 0, 0, 1, 0, 0, 1, 0, 1],
    [1, 0, 1, 0, 1, 1, 1, 1, 1, 0, 1, 1, 0, 1, 1, 1, 1, 0, 1, 0, 0, 1, 0, 1],
    [1, 0, 0, 0, 1, 0, 0, 0, 0, 0, 1, 0, 0, 0, 0, 0, 1, 0, 0, 0, 0, 0, 0, 1],
    [1, 1, 1, 0, 1, 0, 1, 1, 1, 0, 1, 0, 1, 1, 1, 0, 1, 1, 1, 1, 0, 1, 1, 1],
    [1, 0, 0, 0, 0, 0, 1, 0, 0, 0, 0, 0, 0, 0, 1, 0, 0, 0, 0, 0, 0, 0, 0, 1],
    [1, 1, 1, 0, 1, 1, 1, 0, 1, 1, 1, 0, 1, 1, 1, 0, 1, 1, 0, 1, 1, 1, 1, 1],
    [1, 0, 0, 0, 0, 0, 1, 0, 0, 0, 1, 0, 1, 0, 0, 0, 1, 0, 0, 0, 0, 0, 0, 1],
    [1, 0, 1, 1, 1, 0, 1, 1, 1, 0, 1, 0, 1, 0, 1, 1, 1, 1, 1, 1, 0, 1, 0, 1],
    [1, 0, 1, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 1, 0, 1, 0, 1],
    [1, 0, 1, 0, 1, 1, 1, 1, 0, 1, 1, 1, 1, 1, 1, 0, 1, 1, 0, 1, 0, 1, 0, 1],
    [1, 0, 0, 0, 1, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 1, 0, 0, 0, 0, 0, 0, 1],
    [1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1],
]

paredes = []
pontos_validos_chao = []

for l, linha in enumerate(MATRIZ_MAPA):
    for c, valor in enumerate(linha):
        px = c * TAM_BLOCO + TAM_BLOCO // 2
        py = l * TAM_BLOCO + TAM_BLOCO // 2
        if valor == 1:
            paredes.append(pygame.Rect(c * TAM_BLOCO, l * TAM_BLOCO, TAM_BLOCO, TAM_BLOCO))
        else:
            pontos_validos_chao.append((px, py))


class Projetil:
    def __init__(self, x, y, dx, dy, lado, dano=10, tipo='comum'):
        self.x, self.y = x, y
        self.dx, self.dy = dx, dy
        self.velocidade = 7 if tipo != 'dardo' else 9
        self.raio = 3 if tipo == 'comum' else 5
        self.lado = lado
        self.dano = dano
        self.tipo = tipo

    def atualizar(self):
        self.x += self.dx * self.velocidade
        self.y += self.dy * self.velocidade
        rect = pygame.Rect(self.x - self.raio, self.y - self.raio, self.raio * 2, self.raio * 2)
        for parede in paredes:
            if rect.colliderect(parede):
                return False
        return True

    def desenhar(self):
        cor = COR_TIRO
        if self.tipo == 'onda_melee':
            cor = COR_ONDA
        elif self.tipo == 'dardo':
            cor = COR_DARDO
        pygame.draw.circle(tela, cor, (int(self.x), int(self.y)), self.raio)


class MinaTerrestre:
    def __init__(self, x, y, lado):
        self.x, self.y = x, y
        self.lado = lado
        self.raio = 8
        self.dano = 20

    def obter_rect(self):
        return pygame.Rect(self.x - self.raio, self.y - self.raio, self.raio * 2, self.raio * 2)

    def desenhar(self):
        pygame.draw.circle(tela, COR_MINA, (int(self.x), int(self.y)), self.raio, 2)
        pygame.draw.circle(tela, COR_MINA, (int(self.x), int(self.y)), 3)


class Personagem:
    def __init__(self, x, y, cor, time, classe):
        self.x, self.y = x, y
        self.cor = cor
        self.time = time
        self.classe = classe
        self.raio = 15
        self.vida_max = 100
        self.vida = 100
        self.cooldown_tiro = 0

        self.tempo_paralizado = 0
        self.cooldown_ultimate = 0

        if self.classe == 'guerreiro':
            self.velocidade = 2.3
            self.alcance_ataque = 45
            self.dano_base = 20
            self.taxa_ataque = 25
            self.max_cd_ult = 1500
        elif self.classe == 'atirador':
            self.velocidade = 3.2
            self.alcance_ataque = 180
            self.dano_base = 10
            self.taxa_ataque = 35
            self.max_cd_ult = 1500
        elif self.classe == 'suporte':
            self.velocidade = 2.1
            self.alcance_ataque = 200
            self.dano_base = 0
            self.taxa_ataque = 999
            self.max_cd_ult = 1200

        self.destino_x, self.destino_y = x, y
        self.escolher_novo_destino()

    def escolher_novo_destino(self):
        self.destino_x, self.destino_y = random.choice(pontos_validos_chao)

    def obter_rect(self):
        return pygame.Rect(self.x - self.raio, self.y - self.raio, self.raio * 2, self.raio * 2)

    def mover_com_colisao(self, dx, dy):
        if self.tempo_paralizado > 0: return
        if dx == 0 and dy == 0: return

        comprimento = math.hypot(dx, dy)
        dx, dy = dx / comprimento, dy / comprimento

        self.x += dx * self.velocidade
        rect_x = self.obter_rect()
        for parede in paredes:
            if rect_x.colliderect(parede):
                if dx > 0:
                    self.x = parede.left - self.raio - 1
                elif dx < 0:
                    self.x = parede.right + self.raio + 1
                if random.random() < 0.1: self.escolher_novo_destino()
                break

        self.y += dy * self.velocidade
        rect_y = self.obter_rect()
        for parede in paredes:
            if rect_y.colliderect(parede):
                if dy > 0:
                    self.y = parede.top - self.raio - 1
                elif dy < 0:
                    self.y = parede.bottom + self.raio + 1
                if random.random() < 0.1: self.escolher_novo_destino()
                break

    def curar_aliados(self, aliados):
        if self.classe != 'suporte' or self.vida <= 0 or self.tempo_paralizado > 0: return
        for aliado in aliados:
            if aliado != self and aliado.vida > 0:
                dist = math.hypot(aliado.x - self.x, aliado.y - self.y)
                if dist < 120:
                    aliado.vida = min(aliado.vida_max, aliado.vida + 0.3)

    def atacar(self, alvos, lista_tiros):
        if self.tempo_paralizado > 0: return None
        if self.classe == 'suporte': return None
        if self.cooldown_tiro > 0:
            self.cooldown_tiro -= 1
            return None

        alvo_proximo = None
        dist_minima = self.alcance_ataque

        for alvo in alvos:
            if alvo.vida > 0:
                dist = math.hypot(alvo.x - self.x, alvo.y - self.y)
                if dist < dist_minima:
                    dist_minima = dist
                    alvo_proximo = alvo

        if alvo_proximo:
            dx, dy = alvo_proximo.x - self.x, alvo_proximo.y - self.y
            dist = math.hypot(dx, dy)
            if dist > 0:
                if self.classe == 'guerreiro':
                    alvo_proximo.vida -= self.dano_base
                else:
                    lista_tiros.append(Projetil(self.x, self.y, dx / dist, dy / dist, self.time, self.dano_base))
                self.cooldown_tiro = self.taxa_ataque
            return alvo_proximo
        return None

    def usar_ultimate_ia(self, alvos, lista_tiros, lista_minas):
        if self.vida <= 0 or self.tempo_paralizado > 0 or self.cooldown_ultimate > 0: return

        alvo_proximo = None
        dist_minima = 250

        for alvo in alvos:
            if alvo.vida > 0:
                dist = math.hypot(alvo.x - self.x, alvo.y - self.y)
                if dist < dist_minima:
                    dist_minima = dist
                    alvo_proximo = alvo

        if alvo_proximo:
            dx, dy = alvo_proximo.x - self.x, alvo_proximo.y - self.y
            dist = math.hypot(dx, dy)
            if dist > 0:
                if self.classe == 'guerreiro':
                    lista_tiros.append(Projetil(self.x, self.y, dx / dist, dy / dist, self.time, 20, 'onda_melee'))
                    self.cooldown_ultimate = self.max_cd_ult
                elif self.classe == 'atirador':
                    lista_minas.append(MinaTerrestre(self.x, self.y, self.time))
                    self.cooldown_ultimate = self.max_cd_ult
                elif self.classe == 'suporte':
                    lista_tiros.append(Projetil(self.x, self.y, dx / dist, dy / dist, self.time, 20, 'dardo'))
                    self.cooldown_ultimate = self.max_cd_ult

    def desenhar(self):
        if self.tempo_paralizado > 0 and (pygame.time.get_ticks() // 100) % 2 == 0:
            pygame.draw.circle(tela, (100, 200, 255), (int(self.x), int(self.y)), self.raio)
        else:
            pygame.draw.circle(tela, self.cor, (int(self.x), int(self.y)), self.raio)

        if self.classe == 'guerreiro':
            pygame.draw.circle(tela, (0, 0, 0), (int(self.x), int(self.y)), self.raio - 4, 2)
        elif self.classe == 'suporte':
            pygame.draw.circle(tela, (255, 255, 255), (int(self.x), int(self.y)), self.raio - 4, 2)

        largura_barra = 30
        altura_barra = 5
        pygame.draw.rect(tela, (200, 0, 0), (self.x - 15, self.y - 25, largura_barra, altura_barra))
        vida_prop = max(0, self.vida / self.vida_max)
        pygame.draw.rect(tela, (0, 200, 0), (self.x - 15, self.y - 25, largura_barra * vida_prop, altura_barra))


def mostrar_tela_game_over(vitoria=False):
    """Renderiza a camada de Game Over e gerencia os botões clicáveis."""
    # Cria uma superfície transparente para escurecer o fundo do jogo
    sombra = pygame.Surface((LARGURA, ALTURA))
    sombra.set_alpha(180)
    sombra.fill((10, 10, 15))
    tela.blit(sombra, (0, 0))

    fonte_titulo = pygame.font.SysFont(None, 75)
    fonte_botoes = pygame.font.SysFont(None, 28)

    if vitoria:
        texto_titulo = fonte_titulo.render("VITÓRIA DO TIME AZUL!", True, (50, 255, 100))
    else:
        texto_titulo = fonte_titulo.render("GAME OVER - VOCÊ MORREU", True, (255, 50, 50))

    tela.blit(texto_titulo, (LARGURA // 2 - texto_titulo.get_width() // 2, ALTURA // 2 - 80))

    # Desenhar Botão Reiniciar
    pygame.draw.rect(tela, (40, 150, 70), REINICIAR_RECT, border_radius=8)
    pygame.draw.rect(tela, (255, 255, 255), REINICIAR_RECT, 2, border_radius=8)
    txt_reiniciar = fonte_botoes.render("REINICIAR", True, (255, 255, 255))
    tela.blit(txt_reiniciar,
              (REINICIAR_RECT.x + (REINICIAR_RECT.width // 2 - txt_reiniciar.get_width() // 2), REINICIAR_RECT.y + 15))

    # Desenhar Botão Fechar
    pygame.draw.rect(tela, (180, 40, 40), FECHAR_RECT, border_radius=8)
    pygame.draw.rect(tela, (255, 255, 255), FECHAR_RECT, 2, border_radius=8)
    txt_fechar = fonte_botoes.render("FECHAR", True, (255, 255, 255))
    tela.blit(txt_fechar, (FECHAR_RECT.x + (FECHAR_RECT.width // 2 - txt_fechar.get_width() // 2), FECHAR_RECT.y + 15))


def main():
    while True:  # Loop externo para permitir reinicialização completa do estado do jogo
        player = Personagem(75, 75, COR_PLAYER, 'aliado', 'atirador')

        aliados = []
        inimigos = []
        tiros = []
        minas = []

        # Criando Aliados
        aliados.append(Personagem(75, 150, COR_ALIADO, 'aliado', 'guerreiro'))
        aliados.append(Personagem(150, 75, COR_ALIADO, 'aliado', 'guerreiro'))
        aliados.append(Personagem(150, 150, COR_ALIADO, 'aliado', 'atirador'))
        aliados.append(Personagem(75, 220, COR_ALIADO, 'aliado', 'suporte'))

        # Criando Inimigos
        inimigos.append(Personagem(1125, 600, COR_INIMIGO, 'inimigo', 'guerreiro'))
        inimigos.append(Personagem(1050, 675, COR_INIMIGO, 'inimigo', 'guerreiro'))
        inimigos.append(Personagem(1125, 675, COR_INIMIGO, 'inimigo', 'atirador'))
        inimigos.append(Personagem(975, 675, COR_INIMIGO, 'inimigo', 'atirador'))
        inimigos.append(Personagem(1125, 525, COR_INIMIGO, 'inimigo', 'suporte'))

        jogo_ativo = True
        resultado_vitoria = False

        while jogo_ativo:
            relogio.tick(60)

            toque_pressionado = pygame.mouse.get_pressed()[0]
            pos_toque = pygame.mouse.get_pos()

            p_dx, p_dy = 0, 0

            for evento in pygame.event.get():
                if evento.type == pygame.QUIT:
                    pygame.quit()
                    sys.exit()

                if evento.type == pygame.MOUSEBUTTONDOWN:
                    if evento.button == 1:
                        dist_botao = math.hypot(pos_toque[0] - BOTAO_X, pos_toque[1] - BOTAO_Y)
                        if dist_botao <= BOTAO_RAIO:
                            if player.vida > 0 and player.cooldown_ultimate == 0:
                                minas.append(MinaTerrestre(player.x, player.y, 'aliado'))
                                player.cooldown_ultimate = player.max_cd_ult

            if toque_pressionado and player.tempo_paralizado <= 0 and player.vida > 0:
                dist_ao_botao = math.hypot(pos_toque[0] - BOTAO_X, pos_toque[1] - BOTAO_Y)
                if dist_ao_botao > BOTAO_RAIO:
                    dist_ao_dedo = math.hypot(pos_toque[0] - player.x, pos_toque[1] - player.y)
                    if dist_ao_dedo > 10:
                        p_dx = pos_toque[0] - player.x
                        p_dy = pos_toque[1] - player.y
                        player.mover_com_colisao(p_dx, p_dy)

            # Redução dos tempos de recarga
            if player.cooldown_ultimate > 0: player.cooldown_ultimate -= 1
            if player.tempo_paralizado > 0: player.tempo_paralizado -= 1
            for b in aliados + inimigos:
                if b.cooldown_ultimate > 0: b.cooldown_ultimate -= 1
                if b.tempo_paralizado > 0: b.tempo_paralizado -= 1

            todos_aliados = [player] + aliados

            # --- PROCESSAR INTELIGÊNCIA ARTIFICIAL (ALIADOS) ---
            for bot in aliados[:]:
                bot.curar_aliados(todos_aliados)
                bot.usar_ultimate_ia(inimigos, tiros, minas)
                alvo_focado = bot.atacar(inimigos, tiros)

                if bot.tempo_paralizado <= 0:
                    if alvo_focado:
                        dx, dy = alvo_focado.x - bot.x, alvo_focado.y - bot.y
                        dist_parada = 35 if bot.classe == 'guerreiro' else 100
                        if math.hypot(dx, dy) > dist_parada: bot.mover_com_colisao(dx, dy)
                    else:
                        if bot.classe == 'suporte':
                            dx, dy = player.x - bot.x, player.y - bot.y
                            if math.hypot(dx, dy) > 50: bot.mover_com_colisao(dx, dy)
                        else:
                            dx, dy = bot.destino_x - bot.x, bot.destino_y - bot.y
                            if math.hypot(dx, dy) > 15:
                                bot.mover_com_colisao(dx, dy)
                            else:
                                bot.escolher_novo_destino()

            # --- PROCESSAR INTELIGÊNCIA ARTIFICIAL (INIMIGOS) ---
            for bot in inimigos[:]:
                bot.curar_aliados(inimigos)
                bot.usar_ultimate_ia(todos_aliados, tiros, minas)
                alvo_focado = bot.atacar(todos_aliados, tiros)

                if bot.tempo_paralizado <= 0:
                    if alvo_focado:
                        dx, dy = alvo_focado.x - bot.x, alvo_focado.y - bot.y
                        dist_parada = 35 if bot.classe == 'guerreiro' else 100
                        if math.hypot(dx, dy) > dist_parada: bot.mover_com_colisao(dx, dy)
                    else:
                        dx, dy = bot.destino_x - bot.x, bot.destino_y - bot.y
                        if math.hypot(dx, dy) > 15:
                            bot.mover_com_colisao(dx, dy)
                        else:
                            bot.escolher_novo_destino()

            # Ataque automático padrão do Jogador
            if player.vida > 0:
                player.atacar(inimigos, tiros)

            # --- LÓGICA DE DETONAÇÃO DAS MINAS ---
            for mina in minas[:]:
                if mina.lado == 'aliado':
                    for inimigo in inimigos[:]:
                        if mina.obter_rect().colliderect(inimigo.obter_rect()):
                            inimigo.vida -= mina.dano
                            if mina in minas: minas.remove(mina)
                            break
                else:
                    for aliado in todos_aliados:
                        if mina.obter_rect().colliderect(aliado.obter_rect()):
                            aliado.vida -= mina.dano
                            if mina in minas: minas.remove(mina)
                            break

            # --- CHECAGEM DE MORTES ---
            for inimigo in inimigos[:]:
                if inimigo.vida <= 0: inimigos.remove(inimigo)
            for aliado in aliados[:]:
                if aliado.vida <= 0: aliados.remove(aliado)

            # Condição de derrota: jogador perde a vida
            if player.vida <= 0:
                jogo_ativo = False
                resultado_vitoria = False

            # Condição de vitória: eliminar todos os inimigos
            if not inimigos:
                jogo_ativo = False
                resultado_vitoria = True

            # --- PROJÉTEIS ---
            for tiro in tiros[:]:
                if not tiro.atualizar():
                    if tiro in tiros: tiros.remove(tiro)
                    continue
                rect_tiro = pygame.Rect(tiro.x - tiro.raio, tiro.y - tiro.raio, tiro.raio * 2, tiro.raio * 2)

                if tiro.lado == 'aliado':
                    for inimigo in inimigos[:]:
                        if rect_tiro.colliderect(inimigo.obter_rect()):
                            if inimigo.classe == 'suporte' and tiro.tipo != 'onda_melee':
                                tiro.lado = 'inimigo'
                                tiro.dx *= -1
                                tiro.dy *= -1
                            else:
                                inimigo.vida -= tiro.dano
                                if tiro.tipo == 'dardo': inimigo.tempo_paralizado = 120
                                if tiro in tiros: tiros.remove(tiro)
                            break
                else:
                    for aliado in todos_aliados:
                        if rect_tiro.colliderect(aliado.obter_rect()):
                            if aliado.classe == 'suporte' and tiro.tipo != 'onda_melee':
                                tiro.lado = 'aliado'
                                tiro.dx *= -1
                                tiro.dy *= -1
                            else:
                                aliado.vida -= tiro.dano
                                if tiro.tipo == 'dardo': aliado.tempo_paralizado = 120
                                if tiro in tiros: tiros.remove(tiro)
                            break

            # --- RENDERIZAÇÃO PADRÃO DO JOGO ---
            tela.fill(COR_FUNDO)
            for parede in paredes: pygame.draw.rect(tela, COR_PAREDE, parede)
            for m in minas: m.desenhar()
            for tiro in tiros: tiro.desenhar()

            if player.vida > 0: player.desenhar()
            for bot in aliados: bot.desenhar()
            for bot in inimigos: bot.desenhar()

            # Desenhar Botão de Habilidade
            cor_atual_botao = COR_BOTAO if player.cooldown_ultimate == 0 else (60, 60, 65)
            pygame.draw.circle(tela, cor_atual_botao, (BOTAO_X, BOTAO_Y), BOTAO_RAIO)
            pygame.draw.circle(tela, (255, 255, 255), (BOTAO_X, BOTAO_Y), BOTAO_RAIO, 3)
            fonte_botao = pygame.font.SysFont(None, 22)
            if player.cooldown_ultimate == 0:
                txt_btn = fonte_botao.render("MINA", True, (255, 255, 255))
            else:
                txt_btn = fonte_botao.render(f"{int(player.cooldown_ultimate / 60)}s", True, (200, 200, 200))
            tela.blit(txt_btn, (BOTAO_X - 18, BOTAO_Y - 8))

            pygame.display.flip()

        # --- LOOP RETENÇÃO: TELA DE GAME OVER (AGUARDANDO CLIQUE) ---
        tela_game_over_aberta = True
        while tela_game_over_aberta:
            relogio.tick(60)
            mostrar_tela_game_over(vitoria=resultado_vitoria)
            pygame.display.flip()

            for evento in pygame.event.get():
                if evento.type == pygame.QUIT:
                    pygame.quit()
                    sys.exit()

                if evento.type == pygame.MOUSEBUTTONDOWN:
                    if evento.button == 1:
                        pos_clique = pygame.mouse.get_pos()

                        # Verifica se clicou no botão "REINICIAR"
                        if REINICIAR_RECT.collidepoint(pos_clique):
                            tela_game_over_aberta = False  # Sai deste loop secundário e reinicia o main()

                        # Verifica se clicou no botão "FECHAR"
                        elif FECHAR_RECT.collidepoint(pos_clique):
                            pygame.quit()
                            sys.exit()


if __name__ == "__main__":
    main()
