:orphan:

# Release 0.28.0-dev (development release)

<h3>New features since last release</h3>

* Support custom measurement processes:
  * `SampleMeasurement` and `StateMeasurement` classes have been added. They contain an abstract
    method to process samples/quantum state.
    [#3286](https://github.com/PennyLaneAI/pennylane/pull/3286)

  * Add `_Sample` class.
    [#3288](https://github.com/PennyLaneAI/pennylane/pull/3288)

  * Add `_Counts` class.
    [#3292](https://github.com/PennyLaneAI/pennylane/pull/3292)

  * Add `_State` class.
    [#3287](https://github.com/PennyLaneAI/pennylane/pull/3287)

  * Add `_VnEntropy` class.
    [#3326](https://github.com/PennyLaneAI/pennylane/pull/3326)

  * Add `_MutualInfo` class.
    [#3327](https://github.com/PennyLaneAI/pennylane/pull/3327)

* Functionality for fetching symbols and geometry of a compound from the PubChem Database using `qchem.mol_data`.
  [(#3289)](https://github.com/PennyLaneAI/pennylane/pull/3289)
  [(#3378)](https://github.com/PennyLaneAI/pennylane/pull/3378)

  ```pycon
  >>> mol_data("BeH2")
  (['Be', 'H', 'H'],
  array([[ 4.79405604,  0.29290815,  0.        ],
         [ 3.77946   , -0.29290815,  0.        ],
         [ 5.80884105, -0.29290815,  0.        ]]))

  >>> mol_data(223, "CID")
  (['N', 'H', 'H', 'H', 'H'],
  array([[ 4.79404621,  0.        ,  0.        ],
         [ 5.80882913,  0.5858151 ,  0.        ],
         [ 3.77945225, -0.5858151 ,  0.        ],
         [ 4.20823111,  1.01459396,  0.        ],
         [ 5.3798613 , -1.01459396,  0.        ]]))
  ```

* New basis sets, `6-311g` and `CC-PVDZ`, are added to the qchem basis set repo.
  [#3279](https://github.com/PennyLaneAI/pennylane/pull/3279)

* Support for purity computation is added. The `qml.math.purity` function computes the purity from a state vector or a density matrix:

  [#3290](https://github.com/PennyLaneAI/pennylane/pull/3290)

  ```pycon
  >>> x = [1, 0, 0, 1] / np.sqrt(2)
  >>> qml.math.purity(x, [0, 1])
  1.0
  >>> qml.math.purity(x, [0])
  0.5
    
  >>> x = [[1 / 2, 0, 0, 0], [0, 0, 0, 0], [0, 0, 0, 0], [0, 0, 0, 1 / 2]]
  >>> qml.math.purity(x, [0, 1])
  0.5
  ```

  The `qml.qinfo.purity` can be used to transform a QNode returning a state to a function that returns the purity:

  ```python3
  dev = qml.device("default.mixed", wires=2)

  @qml.qnode(dev)
  def circuit(x):
    qml.IsingXX(x, wires=[0, 1])
    return qml.state()
  ```

  ```pycon
  >>> qml.qinfo.purity(circuit, wires=[0])(np.pi / 2)
  0.5
  >>> qml.qinfo.purity(circuit, wires=[0, 1])(np.pi / 2)
  1.0
  ```

  Taking the gradient is also supported:

  ```pycon
  >>> param = np.array(np.pi / 4, requires_grad=True)
  >>> qml.grad(qml.qinfo.purity(circuit, wires=[0]))(param)
  -0.5
  ```

<h3>Improvements</h3>

* Continuous integration checks are now performed for Python 3.11 and Torch v1.13. Python 3.7 is dropped.
  [(#3276)](https://github.com/PennyLaneAI/pennylane/pull/3276)

* `qml.Tracker` now also logs results in `tracker.history` when tracking execution of a circuit.
   [(#3306)](https://github.com/PennyLaneAI/pennylane/pull/3306)

* Improve performance of `Wires.all_wires`.
  [(#3302)](https://github.com/PennyLaneAI/pennylane/pull/3302)

* A representation has been added to the `Molecule` class.
  [#3364](https://github.com/PennyLaneAI/pennylane/pull/3364)

* Remove private `_wires` setter from the `Controlled.map_wires` method.
  [3405](https://github.com/PennyLaneAI/pennylane/pull/3405)

<h3>Breaking changes</h3>

* The `log_base` attribute has been moved from `MeasurementProcess` to the new `_VnEntropy` and
  `_MutualInfo` classes, which inherit from `MeasurementProcess`.
  [#3326](https://github.com/PennyLaneAI/pennylane/pull/3326)

* Python 3.7 support is no longer maintained.
  [(#3276)](https://github.com/PennyLaneAI/pennylane/pull/3276)

<h3>Deprecations</h3>

Deprecations cycles are tracked at [doc/developement/deprecations.rst](https://docs.pennylane.ai/en/latest/development/deprecations.html).

* The following deprecated methods are removed:
  [(#3281)](https://github.com/PennyLaneAI/pennylane/pull/3281/)

  * `qml.tape.get_active_tape`: Use `qml.QueuingManager.active_context()`
  * `qml.transforms.qcut.remap_tape_wires`: Use `qml.map_wires`
  * `qml.tape.QuantumTape.inv()`: Use `qml.tape.QuantumTape.adjoint()`
  * `qml.tape.stop_recording()`: Use `qml.QueuingManager.stop_recording()`
  * `qml.tape.QuantumTape.stop_recording()`: Use `qml.QueuingManager.stop_recording()`
  * `qml.QueuingContext` is now `qml.QueuingManager`
  * `QueuingManager.safe_update_info` and `AnnotatedQueue.safe_update_info`: Use plain `update_info`

<h3>Documentation</h3>

* Corrects more mentions for diagonalizing gates for all relevant operations. The docstrings for `compute_eigvals` used
  to say that the diagonalizing gates implemented $U$, the unitary such that $O = U \Sigma U^{\dagger}$, where $O$ is
  the original observable and $\Sigma$ a diagonal matrix. However, the diagonalizing gates actually implement
  $U^{\dagger}$, since $\langle \psi | O | \psi \rangle = \langle \psi | U \Sigma U^{\dagger} | \psi \rangle$, making
  $U^{\dagger} | \psi \rangle$ the actual state being measured in the $Z$-basis.
  [(#3409)](https://github.com/PennyLaneAI/pennylane/pull/3409)

<h3>Bug fixes</h3>

* Fixed a bug that made `gradients.param_shift` raise an error when used with unshifted terms only
  in a custom recipe, and when using any unshifted terms at all under the new return type system.
  [(#3177)](https://github.com/PennyLaneAI/pennylane/pull/3177)

* Original tape `_obs_sharing_wires` attribute is updated during its expansion.
  [#3293](https://github.com/PennyLaneAI/pennylane/pull/3293)
  
* Small fix of `MeasurementProcess.map_wires`, where both the `self.obs` and `self._wires`
  attributes were modified.
  [#3292](https://github.com/PennyLaneAI/pennylane/pull/3292)

* An issue with `drain=False` in the adaptive optimizer is fixed. Before the fix, the operator pool
  needed to be re-constructed inside the optimization pool when `drain=False`. With the new fix,
  this reconstruction is not needed.
  [#3361](https://github.com/PennyLaneAI/pennylane/pull/3361)

* If the device originally has no shots but finite shots are dynamically specified, Hamiltonian
  expansion now occurs.
  [(#3369)](https://github.com/PennyLaneAI/pennylane/pull/3369)

* `qml.matrix(op)` now fails if the operator truly has no matrix (eg. `Barrier`) to match `op.matrix()`
  [(#3386)](https://github.com/PennyLaneAI/pennylane/pull/3386)

* The `pad_with` argument in the `AmplitudeEmbedding` template is now compatible
  with all interfaces
  [(#3392)](https://github.com/PennyLaneAI/pennylane/pull/3392)

<h3>Contributors</h3>

This release contains contributions from (in alphabetical order):

Juan Miguel Arrazola
Utkarsh Azad
Astral Cai
Pieter Eendebak
Lillian M. A. Frederiksen
Soran Jahangiri
Edward Jiang
Christina Lee
Albert Mitjans Coma
Romain Moyard
David Wierichs
Antal Száva
