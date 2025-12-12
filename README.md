# Arkanoid-Assembly
Implementar o Arkanoid em Assembly foi um desafio técnico relevante. Lidei com detalhes como detecção de colisão, controle por interrupções e atualização de tela em baixo nível. O resultado funcionar corretamente foi satisfatório e trouxe um entendimento mais concreto sobre como o hardware e o software interagem na prática.
;------------------------------------------------------------------------------
; ZONA I: Definicao de constantes
;         Pseudo-instrucao : EQU
;------------------------------------------------------------------------------
CR              EQU     0Ah
FIM_TEXTO       EQU     '@'
IO_READ         EQU     FFFFh
IO_WRITE        EQU     FFFEh
IO_STATUS       EQU     FFFDh
INITIAL_SP      EQU     FDFFh
CURSOR		EQU     FFFCh
CURSOR_INIT	EQU	FFFFh
ROW_POSITION	EQU	0d
COL_POSITION	EQU	0d
ROW_SHIFT	EQU	8d
COLUMN_SHIFT	EQU	8d
PULA_LINHA	EQU	81d
MAPA 		EQU      8000h
LINHA_NAVE	EQU	23d
APAGA           EQU     ' '
BOLA            EQU     'o'
BLOCO           EQU     'X'
PAREDE          EQU     '#'
ZERO            EQU     '0'
DEZ             EQU     10d

TIMER_UNIT	EQU	FFF6H
ACTIVE_TIMER	EQU	FFF7H
OFF		EQU	0d
ON 		EQU     1

DIREITA         EQU      1d
ESQUERDA        EQU     -1d
CIMA            EQU     -1d
BAIXO           EQU      1d
BOLA_LINHA      EQU      22d
BOLA_COLUNA     EQU      26d 
ULTIMA_LINHA    EQU      24d   
FIM_COLUNA      EQU      80d

VITORIA         EQU      100d

;------------------------------------------------------------------------------
; ZONA II: definicao de variaveis
;          Pseudo-instrucoes : WORD - palavra (16 bits)
;                              STR  - sequencia de caracteres (cada ocupa 1 palavra: 16 bits).
;          Cada caracter ocupa 1 palavra
;------------------------------------------------------------------------------
	ORIG 	8000h
L0	STR	'################################################################################', FIM_TEXTO
L1	STR	'# SCORE:00                                                             VIDAS:3 #', FIM_TEXTO
L2	STR	'################################################################################', FIM_TEXTO
L3	STR	'#                                                                              #', FIM_TEXTO
L4	STR	'#                                                                              #', FIM_TEXTO
L5	STR	'#                                                                              #', FIM_TEXTO
L6	STR	'#             XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX       #', FIM_TEXTO
L7	STR	'#             XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX       #', FIM_TEXTO
L8	STR	'#             XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX       #', FIM_TEXTO
L9	STR	'#             XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX       #', FIM_TEXTO
L10	STR	'#                                                                              #', FIM_TEXTO
L11	STR	'#                                                                              #', FIM_TEXTO
L12	STR	'#                                                                              #', FIM_TEXTO
L13	STR	'#                                                                              #', FIM_TEXTO
L14	STR	'#                                                                              #', FIM_TEXTO
L15	STR	'#                                                                              #', FIM_TEXTO
L16	STR	'#                                                                              #', FIM_TEXTO
L17	STR	'#                                                                              #', FIM_TEXTO
L18	STR	'#                                                                              #', FIM_TEXTO
L19	STR	'#                                                                              #', FIM_TEXTO
L20	STR	'#                                                                              #', FIM_TEXTO
L21	STR	'#                                                                              #', FIM_TEXTO
L22	STR	'#                                                                              #', FIM_TEXTO
L23	STR	'                                                                                ', FIM_TEXTO
Vitoria STR     'Voce venceu meu amor!', FIM_TEXTO
Derrota STR     'perdeu otario!', FIM_TEXTO

RowIndex	WORD	0d
ColumnIndex	WORD	0d

Nave_corpo	STR	'####################', FIM_TEXTO
Tamanho_Corpo	WORD	20d
Posicao_NaveI	WORD	27d
Posicao_NaveF	WORD	47d

LinhaBola       WORD    22d
ColunaBola      WORD    40d
Teste           WORD    1d

DirLinha         WORD    1d    ; -1 = sobe, 1 = desce
DirColuna        WORD    1d    ; -1 = esqu, 1 = dir

Vidas           WORD    3d
DezenaTela      STR    '0'
UnidadeTela     STR    '0' 
Pontuacao	WORD	0d
LinhaPontuacao	WORD	1d 
ColunaPontuacao WORD	8d
estadoJogo	WORD	1d  ; 1 = em jogo, 0 = game ove
;------------------------------------------------------------------------------
; ZONA III: definicao de tabela de interrupções
;------------------------------------------------------------------------------
	ORIG    FE00h
INT0    WORD    Esqueda
INT1    WORD    Direita
INT2    WORD    Pause
	                
        ORIG    FE0Fh
INT15	WORD	Timer



;------------------------------------------------------------------------------
; ZONA IV: codigo
;        conjunto de instrucoes Assembly, ordenadas de forma a realizar
;        as funcoes pretendidas
;------------------------------------------------------------------------------
                ORIG    0000h
                JMP     Main

;------------------------------------------------------------------------------  
; Timer
;------------------------------------------------------------------------------  
Timer:  PUSH R1
	PUSH R2
	PUSH R3

        CALL DirecaoBolaY
        CALL DirecaoBolaX
        CALL ConfTimer

	POP R3
	POP R2
	POP R1 
	RTI

;------------------------------------------------------------------------------  
; Timer
;------------------------------------------------------------------------------  
Pause:  PUSH R1

        MOV M[ACTIVE_TIMER], R0
	POP R1 
	RTI

;------------------------------------------------------------------------------
;ConfTimer
;------------------------------------------------------------------------------

ConfTimer:PUSH R1
	PUSH R2
	PUSH R3

	MOV R1, 2d
	MOV M[TIMER_UNIT], R1

	MOV R1, M[estadoJogo]
	MOV	M[ACTIVE_TIMER], R1


	POP R3
	POP R2
	POP R1
	RET 



;------------------------------------------------------------------------------
;Função imprimi mapa
;------------------------------------------------------------------------------

Maprint:    PUSH R1
            PUSH R2
            PUSH R3

            MOV R1, L0          ; Primeira linha
            MOV R2, 0           ; Contador de linhas

cicloMap:   CALL print          ; Imprime linha atual
            INC R2              ; Próxima linha

            CMP R2, ULTIMA_LINHA
            JMP.Z fimMap
            
            ADD R1, PULA_LINHA  ; Próxima linha na memória
            MOV M[RowIndex], R2 ; Atualiza índice da linha

            JMP cicloMap

fimMap:     POP R3
            POP R2
            POP R1
            RET

;------------------------------------------------------------------------------
; Função print para o mapa
;------------------------------------------------------------------------------

print:	                PUSH 	R1	; &String inicial = L0
			PUSH 	R2	
			PUSH 	R3	 
			PUSH 	R4 ; valor do R1


printciclo:	MOV 	R4, M[ R1 ] ; valor do que esta em R1

			CMP 	R4, FIM_TEXTO
			JMP.Z 	Endprint

			MOV R2, M[RowIndex]

			SHL R2,	ROW_SHIFT ; linha
			OR  R2, R3
			MOV M[ CURSOR ], R2
			MOV M[ IO_WRITE ], R4

			INC R3 ; avança para proxima coluna
			INC R1 ; incrementa para a próxima letra da str

			JMP printciclo


Endprint:               POP R4
			POP R3
			POP R2
			POP R1
			RET

;------------------------------------------------------------------------------
;Direita
;------------------------------------------------------------------------------
Direita:PUSH R1
        PUSH R2

        MOV R1, M[Posicao_NaveF]
        CMP R1, 79d          
        JMP.P FimDireita

        MOV R1, LINHA_NAVE       
        SHL R1, ROW_SHIFT
        OR  R1 ,M[Posicao_NaveI] 

        MOV M[CURSOR], R1
        MOV R2, APAGA             
        MOV M[IO_WRITE], R2

        INC M[Posicao_NaveI]
        INC M[Posicao_NaveF]
        
        CALL Imprimenave

FimDireita:POP R2
        POP R1
        RTI


;------------------------------------------------------------------------------
;Esquerda
;------------------------------------------------------------------------------
Esqueda:PUSH R1
        PUSH R2

        MOV R1, M[Posicao_NaveI]
        CMP R1, 1d          
        JMP.N FimEsquerda

        MOV R1, LINHA_NAVE       
        SHL R1, ROW_SHIFT
        OR R1, M[Posicao_NaveF] 
        
        MOV M[CURSOR], R1
        MOV R2, APAGA             
        MOV M[IO_WRITE], R2

        DEC M[Posicao_NaveI]
        DEC M[Posicao_NaveF]
        CALL Imprimenave

FimEsquerda:POP R2
        POP R1
        RTI

;------------------------------------------------------------------------------
;Imprime nave
;------------------------------------------------------------------------------
Imprimenave:    PUSH R1
                PUSH R2
                PUSH R3


                MOV R1, Nave_corpo
                MOV R2, LINHA_NAVE
                MOV M[RowIndex], R2      ; atualiza a linha corrente para print
                
                MOV R3, M[Posicao_NaveI]
                CALL print

                POP R3
                POP R2
                POP R1
                RET

;------------------------------------------------------------------------------
;DirecaoBola
;------------------------------------------------------------------------------
DirecaoBolaY:   PUSH R1

                MOV R1, M[DirLinha]
                CMP R1, CIMA
                CALL.Z MoveCima

                CMP R1, BAIXO
                CALL.Z MoveBaixo

                POP R1
                RET

DirecaoBolaX:   PUSH R1

                MOV R1, M[DirColuna]
                CMP R1, ESQUERDA
                CALL.Z MoveEsquerda

                CMP R1, DIREITA
                CALL.Z MoveDireita

                POP R1
                RET

;------------------------------------------------------------------------------
;Função
;------------------------------------------------------------------------------

MoveCima:       PUSH R1
		PUSH R2
		PUSH R3
                PUSH R4
                PUSH R5
                PUSH R6
                PUSH R7

                
                MOV R1, M[LinhaBola]
                MOV R2, M[ColunaBola]

ColideCima:     DEC R1

                MOV R4, R1         ;R4 = Linha da bola alterado
                MOV R3, PULA_LINHA ;R3 = Tamanho da linha = 81    

                MUL R1, R3
                ADD R3, R2
                ADD R3, MAPA
                
                MOV R5, R3    ;  endereço final do bloco ou da parede
                MOV R3, M[R3]  
                CMP R3, PAREDE
                JMP.Z InverteCima 

                CMP R3, BLOCO
                JMP.Z blocoColideC

ContinuaCima:   CALL apagaBola 
                MOV M[LinhaBola], R4                

FimBolaCima:	CALL PrintBola
                POP R7
                POP R6
                POP R5
                POP R4
                POP R3
		POP R2
		POP R1
		RET 

InverteCima:    MOV R4, BAIXO
                MOV M[DirLinha], R4
                JMP FimBolaCima

blocoColideC:   MOV R7, APAGA
                MOV M[R5], R7
                
                MOV R6, M[LinhaBola]
                DEC R6
                MOV R7, R2 ; coluna
                CALL ApagaBloco
                CALL AumentaScore
                JMP InverteCima


;------------------------------------------------------------------------------
;BAIXO
;------------------------------------------------------------------------------

MoveBaixo:      PUSH R1
		PUSH R2
		PUSH R3
                PUSH R4
                PUSH R5
                PUSH R6
                push R7
                
                
                MOV R1, M[LinhaBola]
                MOV R2, M[ColunaBola]

ColideBaixo:    INC R1
                MOV R5, R1

                CMP R1, LINHA_NAVE
                JMP.Z BateNave 

                MOV R4, R1
                MOV R3, PULA_LINHA

                MUL R1, R3
                ADD R3, R2
                ADD R3, MAPA

                MOV R4, R3
                MOV R3, M[R3]
                CMP R3, 'X'
                JMP.Z blocoColideB


ContinuaBaixo:  CALL apagaBola
                MOV M[LinhaBola], R5            

FimBolaBaixo:	CALL PrintBola
                POP R7
                POP R6
                POP R5
                POP R4
                POP R3
		POP R2
		POP R1
		RET 

InverteBaixo:   MOV R4, CIMA
                MOV M[DirLinha], R4
                JMP FimBolaBaixo

BateNave:       MOV R3, M[Posicao_NaveI]
                MOV R4, M[Posicao_NaveF]
                
                CMP R2, R3
                JMP.N ColideForaNave        ; se coluna < naveI → não colide

                CMP R2, R4
                JMP.P ColideForaNave        
                
                JMP InverteBaixo

ColideForaNave: CALL PerdeVida
                JMP ContinuaBaixo

blocoColideB:   MOV R7, ' '
                MOV M[R4], R7
                
                MOV R6, M[LinhaBola]
                INC R6
                MOV R7, R2
                CALL ApagaBloco
                CALL AumentaScore
                JMP InverteBaixo
;------------------------------------------------------------------------------
;Direita
;------------------------------------------------------------------------------

MoveDireita:    PUSH R1
                PUSH R2
                PUSH R3
                PUSH R4
                PUSH R5
                PUSH R6
                push R7

                MOV R1, M[LinhaBola]
                MOV R2, M[ColunaBola]

ColideDireita:  INC R2
                MOV R4, R2
                MOV R3, PULA_LINHA

                MUL R1, R3
                ADD R3, R2
                ADD R3, MAPA

                MOV R5, R3
                MOV R3, M[R3]
                CMP R3, '#'
                JMP.Z InverteDireita 

                CMP R3, BLOCO
                JMP.Z blocoColideD

                
ContinuaDireita:CALL apagaBola
                MOV M[ColunaBola], R4                

FimBolaDireita: CALL PrintBola

                POP R7
                POP R6
                POP R5
                POP R4
                POP R3
                POP R2
                POP R1
                RET

InverteDireita: MOV R4, ESQUERDA
                MOV M[DirColuna], R4
                JMP FimBolaDireita

blocoColideD:   MOV R7, ' '
                MOV M[R5], R7
                
                MOV R6, M[LinhaBola]
                MOV R7, R2
                CALL ApagaBloco
                CALL AumentaScore
                JMP InverteDireita

;------------------------------------------------------------------------------
;Esquerda
;------------------------------------------------------------------------------

MoveEsquerda:   PUSH R1
                PUSH R2
                PUSH R3
                PUSH R4
                PUSH R5
                PUSH R6
                push R7

                MOV R1, M[LinhaBola]
                MOV R2, M[ColunaBola]

ColideEsquerda: DEC R2
                MOV R3, PULA_LINHA ; tamnho da linha

                MUL R1, R3         ;R1 =  linha x tamnho da linha
                ADD R3, R2         ;R3 =  coluna + tamanho da linha
                ADD R3, MAPA       ;R3 = mapa + (coluna + tamanho da linha)

                MOV R5, R3
                MOV R3, M[R3]
                CMP R3, PAREDE
                JMP.Z InverteEsquerda 

                CMP R3, BLOCO
                JMP.Z blocoColideE                 
                
ContinuaEsquerda:CALL apagaBola
                MOV M[ColunaBola], R2                

FimBolaEsquerda: CALL PrintBola
                
                POP R7
                POP R6
                POP R5
                POP R4
                POP R3
                POP R2
                POP R1
                RET

InverteEsquerda:MOV R4, DIREITA
                MOV M[DirColuna], R4

                JMP FimBolaEsquerda

blocoColideE:   MOV R7, APAGA
                MOV M[R5], R7
                
                MOV R6, M[LinhaBola]
                MOV R7, R2

                CALL ApagaBloco
                CALL AumentaScore
                JMP InverteEsquerda


;------------------------------------------------------------------------------
;PrintBola
;------------------------------------------------------------------------------
PrintBola:      PUSH R1
                PUSH R2

                MOV R1, M[ColunaBola]
                MOV R2, M[LinhaBola]

                SHL R2,	ROW_SHIFT ; linha
		OR  R1, R2 
		MOV M[ CURSOR ], R1

                MOV R2, BOLA
		MOV M[ IO_WRITE ], R2

                POP R2
                POP R1
                RET

;------------------------------------------------------------------------------
;apagaBola
;------------------------------------------------------------------------------
apagaBola:      PUSH R1
                PUSH R2

                MOV R1, M[ColunaBola]
                MOV R2, M[LinhaBola]

                SHL R2,	ROW_SHIFT ; linha
		OR  R1, R2 
		MOV M[ CURSOR ], R1

                MOV R2, APAGA
		MOV M[ IO_WRITE ], R2

                POP R2
                POP R1
                RET

;------------------------------------------------------------------------------
;APAGA BLOCO
;------------------------------------------------------------------------------
ApagaBloco:     PUSH R1
                PUSH R2

                MOV R1, R7
                MOV R2, R6

                SHL R2,	ROW_SHIFT ; linha
		OR  R1, R2 
		MOV M[ CURSOR ], R1

                MOV R2, APAGA
		MOV M[ IO_WRITE ], R2

                POP R2
                POP R1
                RET

;------------------------------------------------------------------------------
;PerdeVida
;------------------------------------------------------------------------------
PerdeVida:      PUSH R1
                PUSH R2
                PUSH R3

                DEC M[Vidas]
                MOV R3, M[Vidas]
                CMP R3, 0d
                CALL.Z Perde
                JMP.Z FimPerdeVida

                MOV R1, 77d
                MOV R2, 1d

                SHL R2,	ROW_SHIFT ; linha
		OR  R1, R2 
		MOV M[ CURSOR ], R1

                MOV R2, R3
                ADD R2, ZERO
		MOV M[ IO_WRITE ], R2

                CALL Reinicia

FimPerdeVida:   POP R3
                POP R2
                POP R1
                RET
;------------------------------------------------------------------------------
;Reinicia
;------------------------------------------------------------------------------
Reinicia:       PUSH R1
                PUSH R2
                
                CALL apagaBola
                MOV R1, BOLA_LINHA
                MOV R2, BOLA_COLUNA
                
                MOV M[LinhaBola], R1
                MOV M[ColunaBola], R2

                MOV R1, CIMA
                MOV M[DirLinha], R1

                CALL Imprimenave

                POP R2
                POP R1
                RET

;------------------------------------------------------------------------------
; AumentaScore
;------------------------------------------------------------------------------
AumentaScore:   PUSH    R1
                PUSH    R2
	        PUSH	R3

                MOV     R1, M[Pontuacao]
                INC     R1
	        MOV     M[Pontuacao], R1
    
	        MOV     R2, DEZ
                DIV     R1, R2           

                ADD     R1, ZERO          
                MOV     M[DezenaTela], R1

                ADD     R2, ZERO          
                MOV     M[UnidadeTela], R2
                CALL    imprimeScore     ; imprime dezena e unidade na tela

                MOV R1, M[Pontuacao]
                CMP R1, VITORIA
                CALL.Z Venceu

                POP 	R3
                POP     R2
                POP     R1
                RET

;------------------------------------------------------------------------------  
; imprimeScore
;------------------------------------------------------------------------------
imprimeScore:   PUSH    R1
                PUSH    R2
                PUSH    R3
                PUSH    R4

                ; imprime dezena
                MOV     R2, M[LinhaPontuacao]
                MOV     R3, M[ColunaPontuacao]
                SHL R2, ROW_SHIFT
                OR      R2, R3                   ; posição da dezena
                
                MOV     M[CURSOR], R2
                MOV     R4, M[DezenaTela]
                MOV     M[IO_WRITE], R4

                ; imprime unidade
                MOV     R2, M[LinhaPontuacao]
                MOV     R3, M[ColunaPontuacao]
                ADD     R3, 1
                SHL     R2, ROW_SHIFT ; unidade está na coluna à direita
                OR     R2, R3

                MOV     M[CURSOR], R2
                MOV     R4, M[UnidadeTela]
                MOV     M[IO_WRITE], R4

                POP     R4
                POP     R3
                POP     R2
                POP     R1
                RET

;------------------------------------------------------------------------------
; VITÓRIA
;------------------------------------------------------------------------------
Venceu: PUSH R1

        MOV R1, 12d            ; linha 12 (exemplo)
        MOV M[RowIndex], R1

        MOV R3, 35d            ; coluna inicial (20 é um exemplo)
                               ; IMPORTANT: print usa R3 como coluna inicial
        MOV R1, Vitoria
        CALL print

        ; desliga timer para "congelar" jogo (opcional)
        MOV R1, OFF
        MOV M[estadoJogo], R1
        POP R1
        RET
;------------------------------------------------------------------------------
; DERROTA
;------------------------------------------------------------------------------
Perde:PUSH R1

        ; linha e coluna da mensagem
        MOV R1, 12d            ; linha no meio da tela
        MOV M[RowIndex], R1

        MOV R3, 35d            ; coluna inicial
        MOV R1, Derrota     ; endereço da string
        CALL print


        MOV R1, OFF
        MOV M[estadoJogo], R1
        POP R1
        RET


Main:ENI

	MOV		R1, INITIAL_SP
	MOV		SP, R1		 		; We need to initialize the stack
	MOV		R1, CURSOR_INIT		; We need to initialize the cursor 
	MOV		M[ CURSOR ], R1		; with value CURSOR_INIT

        
	CALL Maprint
        CALL Imprimenave
        CALL PrintBola
        

        CALL ConfTimer

        
	

;./p3as-win arkanoidfim.as; java -jar p3sim.jar arkanoidfim.exe
Cycle: 			BR		Cycle	
Halt:           BR		Halt
