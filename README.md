# dynamic-matrix-input
Action for building a matrix variable from a comma-separated string input

## Example:
The two blocks below are an example of the required blocks for the matrixing to function. 

The list that you want your job to be matrixed over must be given as a comma separated list in a string. 

Workflow input
```
    inputs:
      ...  
      my_input:  
        description: 'Matrix input (comma-separated)'  
        required: true  
        default: 'Hello,World,This,Is,A,Test'  
        type: string  
      ...  
```

First, the input must then be given to the dynamic-matrix-input action using the `with` directive and with the name `matrix_input`

Next, the output of your matrix setup job must then retrieve the json list using the the following syntax `steps.[name of setup job].outputs.matrix`

Required Job
```
setup_matrix:
    runs-on: ubuntu-latest
    outputs:
      matrix: ${{ steps.setup_matrix.outputs.matrix_output }}
    steps:
      - id: setup_matrix
        uses: cschadewitz/dynamic-matrix@v1
        with:
          matrix_input: ${{ github.event.inputs.my_input }}
```

Finally, to execute a matrixed job over this new json list you must pass it as a matrix variable using the `fromJson` expression. Additionally, the matrixed job must depend on the setup job using the `needs` directive

```
  test_matrix:
    needs: setup_matrix
    runs-on: ubuntu-latest
    strategy:
      matrix: 
        word: ${{ fromJson(needs.setup_matrix.outputs.matrix) }}
    steps:
      - name: Test matrix
        run: echo ${{ matrix.word }}
```

