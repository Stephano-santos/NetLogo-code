turtles-own [
  has-information
  group-id
]

globals [
  number-of-groups
  initial-student-id ;; Variável global para armazenar o ID do aluno inicial
]

to setup
  clear-all
  ;; Definir barreiras
  ask patches with [pxcor > -5 and pxcor < 5 and pycor > -5 and pycor < 5] [
    set pcolor gray
  ]
  set number-of-groups 5  ;; Definir o número de grupos
  
  ;; Definir cor de fundo
  set-patch-size 20
  ask patches [ set pcolor black ]  ;; Cor de fundo
  
  ;; Criar alunos
  create-turtles 30 [
    set shape "person"
    set color blue  ;; Cor dos alunos
    set size 1.2
    setxy random-xcor random-ycor
    set has-information false
    set group-id 0  ;; Inicializar grupo como 0
  ]
  
  ;; Criar professor
  create-turtles 1 [
    set shape "person"
    set color red  ;; Cor do professor definida como vermelho
    set size 2
    setxy random-xcor random-ycor
    set has-information false ;; Garantir que o professor também inicializa com has-information
  ]
  
  ;; Escolher um aluno inicial para começar com a informação
  let initial-student one-of turtles with [ color != red ]
  ask initial-student [
    set has-information true
    set initial-student-id who ;; Armazenar o ID do aluno inicial
  ]
  
  ;; Organizar alunos em grupos coesos
  assign-groups
  
  reset-ticks
end

to go
  ;; Movimentação dos alunos
  ask turtles with [ color != red ] [
    move-in-group
    if has-information and (who = initial-student-id) [ ;; Permitir que apenas o aluno inicial passe a informação
      pass-information
    ]
  ]
  
  ;; Movimentação do professor
  ask turtles with [ color = red ] [
    move-professor
    check-cheating
  ]
  
  ;; Verificar condições de término do jogo
  let students-with-information count turtles with [ has-information ]
  if students-with-information = count turtles with [ color != red ] [
    user-message "Todos os alunos receberam cola! Fim do jogo."
    stop
  ]
  
  tick
end

to assign-groups
  ;; Agrupar os alunos em bandos (grupos coesos)
  let current-group-id 1
  let students-left turtles with [ color != red ]
  
  while [ any? students-left ] [
    let student one-of students-left
    ask student [
      if group-id = 0 [
        set group-id current-group-id
        let my-neighbors other students-left in-radius 3
        ask my-neighbors [
          if group-id = 0 [
            set group-id current-group-id
          ]
        ]
        set students-left turtles with [ color != red and group-id = 0 ]
      ]
    ]
    set current-group-id current-group-id + 1
  ]
end

to move-in-group
  ;; Movimenta os alunos dentro de seu grupo (bando)
  fd random-float 1.5
  if random 100 < 10 [
    rt random 60 - 30
  ]
end

to move-professor
  ;; Movimenta o professor aleatoriamente
  fd 0.85 ;; Aumentamos a velocidade de movimento do professor
  if random 100 < 20 [ ;; Aumentamos a probabilidade de mudar de direção
    rt random 60 - 30
  ]
end

to check-cheating
  ;; Verifica se o professor pegou o aluno inicial colando
  if any? other turtles-here with [ has-information and who = initial-student-id ] [
    user-message "Professor pegou o aluno passando cola! Fim do jogo."
    stop
  ]
end

to pass-information
  ;; Passa informação para outro aluno próximo
  let target one-of other turtles with [ color != red and not has-information and distance myself < 2.1 ]
  if target != nobody and random 100 < 30 [
    ask target [
      set has-information true
    ]
  ]
end
