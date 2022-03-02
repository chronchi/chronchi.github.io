---
layout: post
---

Função que encontra o pivô da coluna, em outras palavras, o lowest one.


```julia
function get_low(column)
    biggest_value = 0
    for j = 1:(length(column))
        if column[j] == 1
            biggest_value = j
        end
    end
    return biggest_value
end   
```

Algoritmo para redução
---


```julia
function reduce_matrix(boundary)

    nb_simplex = size(boundary)[1]

    reduced = boundary

    cycles = one(boundary)

    for col in 1:nb_simplex
        lowest_ones = [get_low(reduced[:,k]) for k in 1:(col-1)]
        while sum(get_low(reduced[:,col]) .== lowest_ones) != 0 && get_low(reduced[:,col]) != 0
            for k in 1:length(lowest_ones)
                if get_low(reduced[:,k]) == get_low(reduced[:,col])
                    reduced[:,col] = rem.(reduced[:,col] + reduced[:,k], 2)
                    cycles[:,col]  = rem.(cycles[:,col] + cycles[:,k], 2)
                end
            end
        end
    end

    return reduced, cycles
end
```

Exemplo
---


```julia
using DelimitedFiles

boundary = readdlm("boundary.txt")

boundary
```




    7×7 Array{Float64,2}:
     0.0  0.0  0.0  1.0  1.0  0.0  0.0
     0.0  0.0  0.0  1.0  0.0  1.0  0.0
     0.0  0.0  0.0  0.0  1.0  1.0  0.0
     0.0  0.0  0.0  0.0  0.0  0.0  1.0
     0.0  0.0  0.0  0.0  0.0  0.0  1.0
     0.0  0.0  0.0  0.0  0.0  0.0  1.0
     0.0  0.0  0.0  0.0  0.0  0.0  0.0



Calculando a matriz reduzida do bordo acima


```julia
@time reduced, cycles = reduce_matrix(boundary)

reduced
```

      0.195500 seconds (198.82 k allocations: 8.872 MiB)





    7×7 Array{Float64,2}:
     0.0  0.0  0.0  1.0  1.0  0.0  0.0
     0.0  0.0  0.0  1.0  0.0  0.0  0.0
     0.0  0.0  0.0  0.0  1.0  0.0  0.0
     0.0  0.0  0.0  0.0  0.0  0.0  1.0
     0.0  0.0  0.0  0.0  0.0  0.0  1.0
     0.0  0.0  0.0  0.0  0.0  0.0  1.0
     0.0  0.0  0.0  0.0  0.0  0.0  0.0



Outro exemplo (outro triângulo)
--


```julia
boundary_2 = readdlm("boundary_2.txt")
```




    7×7 Array{Float64,2}:
     0.0  0.0  0.0  1.0  0.0  1.0  0.0
     0.0  0.0  0.0  1.0  1.0  0.0  0.0
     0.0  0.0  0.0  0.0  1.0  1.0  0.0
     0.0  0.0  0.0  0.0  0.0  0.0  1.0
     0.0  0.0  0.0  0.0  0.0  0.0  1.0
     0.0  0.0  0.0  0.0  0.0  0.0  1.0
     0.0  0.0  0.0  0.0  0.0  0.0  0.0




```julia
reduced_2, cycles_2 = reduce_matrix(boundary_2)

reduced_2
```




    7×7 Array{Float64,2}:
     0.0  0.0  0.0  1.0  0.0  0.0  0.0
     0.0  0.0  0.0  1.0  1.0  0.0  0.0
     0.0  0.0  0.0  0.0  1.0  0.0  0.0
     0.0  0.0  0.0  0.0  0.0  0.0  1.0
     0.0  0.0  0.0  0.0  0.0  0.0  1.0
     0.0  0.0  0.0  0.0  0.0  0.0  1.0
     0.0  0.0  0.0  0.0  0.0  0.0  0.0




```julia
cycles_2
```




    7×7 Array{Float64,2}:
     1.0  1.0  0.0  0.0  0.0  0.0  0.0
     0.0  1.0  1.0  0.0  0.0  0.0  0.0
     0.0  0.0  1.0  0.0  0.0  0.0  0.0
     0.0  0.0  0.0  1.0  0.0  1.0  0.0
     0.0  0.0  0.0  0.0  1.0  1.0  0.0
     0.0  0.0  0.0  0.0  0.0  1.0  0.0
     0.0  0.0  0.0  0.0  0.0  0.0  1.0




```julia
boundary_3 = readdlm("boundary_3.txt")

@time reduced_3, cycles_3 = reduce_matrix(boundary_3)

reduced_3
```

      0.000078 seconds (310 allocations: 117.031 KiB)





    13×13 Array{Float64,2}:
     0.0  0.0  0.0  0.0  0.0  0.0  1.0  0.0  0.0  0.0  0.0  0.0  0.0
     0.0  0.0  0.0  0.0  0.0  0.0  1.0  1.0  0.0  0.0  0.0  0.0  0.0
     0.0  0.0  0.0  0.0  0.0  0.0  0.0  0.0  0.0  1.0  1.0  0.0  0.0
     0.0  0.0  0.0  0.0  0.0  0.0  0.0  0.0  0.0  0.0  1.0  0.0  0.0
     0.0  0.0  0.0  0.0  0.0  0.0  0.0  0.0  0.0  1.0  0.0  0.0  0.0
     0.0  0.0  0.0  0.0  0.0  0.0  0.0  1.0  0.0  0.0  0.0  0.0  0.0
     0.0  0.0  0.0  0.0  0.0  0.0  0.0  0.0  0.0  0.0  0.0  0.0  0.0
     0.0  0.0  0.0  0.0  0.0  0.0  0.0  0.0  0.0  0.0  0.0  0.0  0.0
     0.0  0.0  0.0  0.0  0.0  0.0  0.0  0.0  0.0  0.0  0.0  0.0  0.0
     0.0  0.0  0.0  0.0  0.0  0.0  0.0  0.0  0.0  0.0  0.0  0.0  1.0
     0.0  0.0  0.0  0.0  0.0  0.0  0.0  0.0  0.0  0.0  0.0  0.0  1.0
     0.0  0.0  0.0  0.0  0.0  0.0  0.0  0.0  0.0  0.0  0.0  0.0  1.0
     0.0  0.0  0.0  0.0  0.0  0.0  0.0  0.0  0.0  0.0  0.0  0.0  0.0



Encontrando os diagramas de persistência
--
**Pareamento**

Vamos parear os simplexos em relação ao nascimento e morte de ciclo.


```julia
function simplex_pairing(reduced)
    # primeiro vamos parear as colunas não nulas
    pairs = []
    nb_simplex = size(reduced)[1]
    for j in 1:nb_simplex
        temp = reduced[:,j]
        if temp != zeros(nb_simplex)
            lowest = get_low(temp)
            pairs = [pairs; (lowest, j)]
        end
    end

    # agora as colunas nulas que não foram pareadas são pareadas com o infinito
    for j in 1:nb_simplex
        if sum(j .== [pairs[i][2] for i in 1:length(pairs)]) == 0
            if reduced[j] == 0 && sum(j .== [pairs[i][1] for i in 1:length(pairs)]) == 0
                pairs = [pairs; (j, Inf)]
            end
        end
    end
    return pairs
end
```

Pareamento para as matrizes reduzidas que obtivemos antes


```julia
pairs_1 = simplex_pairing(reduced)

println(pairs_1)

pairs_2 = simplex_pairing(reduced_2)

println(pairs_2)

pairs_3 = simplex_pairing(reduced_3)

println(pairs_3)
```

    Any[(2, 4), (3, 5), (6, 7), (1, Inf)]
    Any[(2, 4), (3, 5), (6, 7), (1, Inf)]
    Any[(2, 7), (6, 8), (5, 10), (4, 11), (12, 13), (1, Inf), (3, Inf), (9, Inf)]
