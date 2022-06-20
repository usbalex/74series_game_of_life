# 74series Game of Life
Submission for 74xx design challenge https://twitter.com/brouhaha/status/1537581442288746498

# Circuit structure/description:
Circuit for computing the state variable of a single cell in [Conway's Game of Life](https://en.wikipedia.org/wiki/Conway%27s_Game_of_Life).
This design uses 3 logic chips. The inputs and outputs use positive logic (1 == alive, 0 == dead).

## Inputs:
  - 8 bits for surrounding cell states/neighbours (n0 to n7)
  - 1 bit  for current cell state (x)
## Outputs:
  - 1 bit  for next cell state (z)

## U1: SN74LS283 (4 bit binary adder with carry in/out)
Sums up n1 to n3 and n4 to n6 independently using bitslicing.

Connections:
  * C0 = n1
  * A1 = n2
  * B1 = n3  
        => sum_a = <S2:S1> = n1+n2+n3
  * A2 = 0
  * B2 = 0  
        => S2 = 0 AND internal carry S2 = 0
  * A3 = n4
  * B3 = n4  
        => internal carry C3 = n3
  * A4 = n5
  * B4 = n6  
        => sum_b = <C4:S4> = n4+n5+n6


## U2: SN74LS283 (4 bit binary adder with carry in/out)
Adds up the intermediate values sum_a and sum_b together with n7

Connections
  * C0 = n7
  * A1 = sum_a<1>
  * B1 = sum_b<1>
  * A2 = sum_a<2>
  * B2 = sum_b<2>
  * A3 = 0
  * B3 = 0  
        => sum_c = <S3:S1> = (n1+n2+n3)+(n4+n5+n6)+n7
  * A4 = 0 / d.c.
  * B4 = 0 / d.c.


## U3: 74HC(T)151 (8-to-1 MUX with active low enable)

Connections:
  * /G (/E) = sum_c<3>
  * C  (S2) = sum_c<2>
  * B  (S1) = sum_c<1>
  * A  (S0) = n8
  * D0 (I0) = 0
  * D1 (I1) = 0
  * D2 (I2) = 0
  * D3 (I3) = x (current_state)
  * D4 (I4) = x (current_state)
  * D5 (I5) = 1
  * D6 (I6) = 1
  * D7 (I7) = 0
  * Y       = z (next_state)
  * W  (/Y) = n.c.

Resulting truth table: (N == number of living neighbour(s) n1+...+n8)

        /G C B A | selected input | output | comment
        ---------|----------------|--------|--------
         1 . . . | n.a / d.c.     | 0      | N > 3 -> dead (overpopulation)
         0 0 0 0 | D0 = 0         | 0      | N = 0 -> dead (underpopulation)
         0 0 0 1 | D1 = 0         | 0      | N = 1 -> dead (underpopulation)
         0 0 1 0 | D2 = 0         | 0      | N = 1 -> dead (underpopulation)
         0 0 1 1 | D3 = x         | x      | N = 2 -> previous state (lives on / stays dead)
         0 1 0 0 | D4 = x         | x      | N = 2 -> previous state (lives on / stays dead)
         0 1 0 1 | D5 = 1         | 1      | N = 3 -> alive (lives on / becomes alive)
         0 1 1 0 | D6 = 1         | 1      | N = 3 -> alive (lives on / becomes alive)
         0 1 1 1 | D7 = 0         | 0      | N = 4 -> dead (overpopulation)
