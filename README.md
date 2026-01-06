#include <stdio.h>
#include <stdlib.h> // Para a função abs

#define BOARD_SIZE 10   // Tamanho fixo do tabuleiro 10x10
#define SHIP_SIZE 3     // Tamanho fixo dos navios (3 posições)
#define SKILL_SIZE 5    // Tamanho fixo das matrizes de habilidades (5x5)

// Função principal do programa
int main() {
    // Declaração e inicialização do tabuleiro como matriz 10x10 com todos os valores 0 (água)
    int board[BOARD_SIZE][BOARD_SIZE];
    for (int i = 0; i < BOARD_SIZE; i++) {
        for (int j = 0; j < BOARD_SIZE; j++) {
            board[i][j] = 0; // Inicializa cada posição com 0
        }
    }

    // Posicionamento dos navios: quatro navios de tamanho 3, dois horizontais/verticais e dois diagonais
    // Cada posicionamento é validado para estar dentro dos limites e sem sobreposição

    // Navio horizontal: linha 1, colunas 0 a 2
    int hor_r = 1, hor_c = 0;
    int can_place = 1; // Flag para validar posicionamento
    for (int k = 0; k < SHIP_SIZE; k++) {
        if (hor_c + k >= BOARD_SIZE || board[hor_r][hor_c + k] != 0) {
            can_place = 0; // Invalida se fora dos limites ou ocupado
            break;
        }
    }
    if (can_place) {
        for (int k = 0; k < SHIP_SIZE; k++) {
            board[hor_r][hor_c + k] = 3; // Coloca o navio com valor 3
        }
    } // Se não puder colocar, ignora (simplificação)

    // Navio vertical: linhas 3 a 5, coluna 2
    int ver_r = 3, ver_c = 2;
    can_place = 1;
    for (int k = 0; k < SHIP_SIZE; k++) {
        if (ver_r + k >= BOARD_SIZE || board[ver_r + k][ver_c] != 0) {
            can_place = 0;
            break;
        }
    }
    if (can_place) {
        for (int k = 0; k < SHIP_SIZE; k++) {
            board[ver_r + k][ver_c] = 3;
        }
    }

    // Navio diagonal crescente (/): linhas 0 a 2, colunas 7 a 9
    int diag1_r = 0, diag1_c = 7;
    can_place = 1;
    for (int k = 0; k < SHIP_SIZE; k++) {
        if (diag1_r + k >= BOARD_SIZE || diag1_c + k >= BOARD_SIZE || board[diag1_r + k][diag1_c + k] != 0) {
            can_place = 0;
            break;
        }
    }
    if (can_place) {
        for (int k = 0; k < SHIP_SIZE; k++) {
            board[diag1_r + k][diag1_c + k] = 3;
        }
    }

    // Navio diagonal decrescente (\): linhas 0 a 2, colunas 9 a 7
    int diag2_r = 0, diag2_c = 9;
    can_place = 1;
    for (int k = 0; k < SHIP_SIZE; k++) {
        if (diag2_r + k >= BOARD_SIZE || diag2_c - k < 0 || board[diag2_r + k][diag2_c - k] != 0) {
            can_place = 0;
            break;
        }
    }
    if (can_place) {
        for (int k = 0; k < SHIP_SIZE; k++) {
            board[diag2_r + k][diag2_c - k] = 3;
        }
    }

    // Criação das matrizes de habilidades: cone, cruz e octaedro (losango)
    // Cada matriz é 5x5, construída dinamicamente com loops aninhados e condicionais
    // 1 indica área afetada, 0 não afetada

    // Matriz para habilidade Cone: forma de cone apontando para baixo, widening from top
    int cone[SKILL_SIZE][SKILL_SIZE];
    for (int i = 0; i < SKILL_SIZE; i++) { // Loop pelas linhas
        for (int j = 0; j < SKILL_SIZE; j++) { // Loop pelas colunas
            // Condicional: área afetada se j estiver no intervalo que se expande com i (do centro para as laterais)
            if (j >= (SKILL_SIZE / 2 - i) && j <= (SKILL_SIZE / 2 + i)) {
                cone[i][j] = 1;
            } else {
                cone[i][j] = 0;
            }
        }
    }

    // Matriz para habilidade Cruz: linhas e colunas centrais
    int cross[SKILL_SIZE][SKILL_SIZE];
    for (int i = 0; i < SKILL_SIZE; i++) {
        for (int j = 0; j < SKILL_SIZE; j++) {
            // Condicional: afetado se na linha central ou coluna central
            if (i == SKILL_SIZE / 2 || j == SKILL_SIZE / 2) {
                cross[i][j] = 1;
            } else {
                cross[i][j] = 0;
            }
        }
    }

    // Matriz para habilidade Octaedro (losango): distância de Manhattan <= raio
    int octa[SKILL_SIZE][SKILL_SIZE];
    for (int i = 0; i < SKILL_SIZE; i++) {
        for (int j = 0; j < SKILL_SIZE; j++) {
            // Condicional: afetado se |i - centro| + |j - centro| <= raio (SKILL_SIZE/2)
            if (abs(i - SKILL_SIZE / 2) + abs(j - SKILL_SIZE / 2) <= SKILL_SIZE / 2) {
                octa[i][j] = 1;
            } else {
                octa[i][j] = 0;
            }
        }
    }

    // Integração das habilidades ao tabuleiro: sobreposição das áreas de efeito
    // Para cada habilidade, define ponto de origem e sobrepõe, marcando com 5 se dentro dos limites

    // Habilidade Cone: origem no topo (ponto superior central), posição board[4][4]
    int cone_origin_row = 4;
    int cone_origin_col = 4;
    for (int di = 0; di < SKILL_SIZE; di++) { // Loop pelas linhas da matriz de habilidade
        for (int dj = 0; dj < SKILL_SIZE; dj++) { // Loop pelas colunas
            if (cone[di][dj] == 1) { // Apenas posições afetadas
                // Calcula posição no tabuleiro: row aumenta para baixo, col ajustado do centro
                int br = cone_origin_row + di;
                int bc = cone_origin_col + (dj - SKILL_SIZE / 2);
                // Condicional para verificar limites do tabuleiro
                if (br >= 0 && br < BOARD_SIZE && bc >= 0 && bc < BOARD_SIZE) {
                    board[br][bc] = 5; // Marca área afetada com 5 (sobrepõe navios ou água)
                }
            }
        }
    }

    // Habilidade Cruz: origem no centro, posição board[2][5]
    int cross_origin_row = 2;
    int cross_origin_col = 5;
    for (int di = 0; di < SKILL_SIZE; di++) {
        for (int dj = 0; dj < SKILL_SIZE; dj++) {
            if (cross[di][dj] == 1) {
                // Calcula posição: ajustado do centro da matriz
                int br = cross_origin_row + (di - SKILL_SIZE / 2);
                int bc = cross_origin_col + (dj - SKILL_SIZE / 2);
                if (br >= 0 && br < BOARD_SIZE && bc >= 0 && bc < BOARD_SIZE) {
                    board[br][bc] = 5;
                }
            }
        }
    }

    // Habilidade Octaedro: origem no centro, posição board[7][7]
    int octa_origin_row = 7;
    int octa_origin_col = 7;
    for (int di = 0; di < SKILL_SIZE; di++) {
        for (int dj = 0; dj < SKILL_SIZE; dj++) {
            if (octa[di][dj] == 1) {
                int br = octa_origin_row + (di - SKILL_SIZE / 2);
                int bc = octa_origin_col + (dj - SKILL_SIZE / 2);
                if (br >= 0 && br < BOARD_SIZE && bc >= 0 && bc < BOARD_SIZE) {
                    board[br][bc] = 5;
                }
            }
        }
    }

    // Exibição do tabuleiro: loops aninhados para imprimir a matriz com espaços para legibilidade
    for (int i = 0; i < BOARD_SIZE; i++) { // Loop pelas linhas
        for (int j = 0; j < BOARD_SIZE; j++) { // Loop pelas colunas
            printf("%d ", board[i][j]); // Imprime valor (0: água, 3: navio, 5: área afetada)
        }
        printf("\n"); // Nova linha após cada linha do tabuleiro
    }

    return 0; // Fim do programa
}
