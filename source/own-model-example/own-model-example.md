---
authors:
  - name: Jonas Schwab
    email: jonas.schwab@uni-wuerzburg.de
    orcid: 0000-0003-3794-8631
---

# How to build your own model

On the example of Dirac fermions coupled to an Ising model

## Introduction

### The Physics

Nematic quantum criticality in Dirac systems: {cite}`Schwab22`.

![image](test.pdf)

- Nematic phase transition of Dirac Fermions

  - Break rotation symmetry without opening a gap\
    $\Rightarrow$ Allow meandering of Dirac cones

- Continuous transitions

- No emergent Lorentz symmetry

- Very slow RG-flow\
  $\Rightarrow$ Numerical (and experimental) results do **not**
  represent IR fixed point\
  $\bullet$ Quasiuniversality

### The Model

$$\mathcal{H} = \mathcal{H}_0 + \mathcal{H}_\text{Ising} + \mathcal{H}_\text{Yuk}$$

- Non-interacting Dirac fermions

  - $\pi$-flux square lattice

- Transverse-field Ising model\
  $\mathcal{H}_\text{Ising} = -J \sum_{\langle \boldsymbol{R},\boldsymbol{R}' \rangle} \hat{s}_{\boldsymbol{R}}^z \hat{s}_{\boldsymbol{R}'}^z -h \sum_{\boldsymbol{R}} \hat{s}_{\boldsymbol{R}}^x$

- Yukawa interaction

  - Couple fermions to spins through terms of form
    $\alpha \hat{s}_{\boldsymbol{R}}^z \hat{c}^\dag_{\boldsymbol{R}} \hat{c}_{\boldsymbol{R}+\boldsymbol{\delta}}$

![image](latt1_0.pdf)

![image](latt1_Ising.pdf)

![image](latt1_Yuk.pdf)

$$\begin{aligned}
\hat{\mathcal{H}}
&=
\hat{\mathcal{H}}_{T}+\hat{\mathcal{H}}_{V} +  \hat{\mathcal{H}}_{I} +   \hat{\mathcal{H}}_{0,I}\,,\;\text{where}
\\
\hat{\mathcal{H}}_{T}
&=
\sum\limits_{k=1}^{M_T}
\sum\limits_{\sigma=1}^{N_{\mathrm{col}}}
\sum\limits_{s=1}^{N_{\mathrm{fl}}}
\sum\limits_{x,y}^{N_{\mathrm{dim}}}
\hat{c}^{\dagger}_{x \sigma   s}T_{xy}^{(k s)} \hat{c}^{\phantom\dagger}_{y \sigma s}  \equiv  \sum\limits_{k=1}^{M_T} \hat{T}^{(k)}
\,,\\
\hat{\mathcal{H}}_{V}
&=
\sum\limits_{k=1}^{M_V}U_{k}
\left\{
\sum\limits_{\sigma=1}^{N_{\mathrm{col}}}
\sum\limits_{s=1}^{N_{\mathrm{fl}}}
\left[
\left(
\sum\limits_{x,y}^{N_{\mathrm{dim}}}
\hat{c}^{\dagger}_{x \sigma s}V_{xy}^{(k s)}\hat{c}^{\phantom\dagger}_{y \sigma s}
\right)
+\alpha_{k s} 
\right]
\right\}^{2}  \equiv   
\sum\limits_{k=1}^{M_V}U_{k}   \left(\hat{V}^{(k)} \right)^2
\,,\\
\hat{\mathcal{H}}_{I}
& = 
\sum\limits_{k=1}^{M_I} \hat{Z}_{k} 
\left(
\sum\limits_{\sigma=1}^{N_{\mathrm{col}}}
\sum\limits_{s=1}^{N_{\mathrm{fl}}}
\sum\limits_{x,y}^{N_{\mathrm{dim}}}
\hat{c}^{\dagger}_{x \sigma s} I_{xy}^{(k s)}\hat{c}^{\phantom\dagger}_{y \sigma s}
\right) \equiv \sum\limits_{k=1}^{M_I} \hat{Z}_{k}    \hat{I}^{(k)} 
\,.
\end{aligned}$$

$$\begin{aligned}
\mathcal{H}_0 &= \hat{\mathcal{H}}_{T}
&
\mathcal{H}_\text{Yuk} &= \hat{\mathcal{H}}_{I}
&
\mathcal{H}_\text{Ising} &= \hat{\mathcal{H}}_{0,I}
&
\end{aligned}$$

## Write Hamiltonian

To Do Goal: Write a new Hamiltonian called `Nematic_Dirac_demo`

Steps necessary:

1.  Create `Hamiltonian_Nematic_Dirac_demo_smod.F90` in
    `Prog/Hamiltonians/`

    - Define parameters

    - Create lattice

    - Set up hopping

    - Set up interaction

      - `Op_V`

      - `S0`

    - Define observables

2.  Add `Nematic_Dirac_demo` to `Prog/Hamiltonians.list`

Copy template

```sh
cp Hamiltonian_##NAME##_smod.F90 Hamiltonian_Nematic_Dirac_demo_smod.F90
```

Replace all occurrences of `##NAME##` with `Nematic_Dirac_demo`

Add optional procedure `S0` to enable Ising dynamics
```fortran
        procedure, nopass :: S0
```

Define parameters Parameter specification:

- Starts with `#PARAMETERS START# <namelist name>`
- Ends with `#PARAMETERS END#`
- One parameter per line, with default value
- Optional: Description

```fortran
      !#PARAMETERS START# VAR_Nematic_Dirac
      !Integer :: N_SUN = 2       ! SU(N) symmetry
      real(Kind=Kind(0.d0)) :: Dtau = 0.1d0    ! Imaginary time step size
      real(Kind=Kind(0.d0)) :: beta = 10.d0    ! Reciprocal temperature
      Integer  :: L1 = 4          ! Size of lattice in a1 direction
      Integer  :: L2 = 4          ! Size of lattice in a2 direction
      real(Kind=Kind(0.d0)) :: ham_t = 1.d0    ! Hopping amplitude of fermions
      real(Kind=Kind(0.d0)) :: Ham_h = 3.d0    ! Ising transverse field
      real(Kind=Kind(0.d0)) :: Ham_J = 1.d0    ! Ferromagnetic Ising interaction
      real(Kind=Kind(0.d0)) :: Ham_xi = 1.d0   ! Coupling Ising spins <-> fermions
      real(Kind=Kind(0.d0)) :: Ham_chem = 0.d0 ! Chemical potential
      !#PARAMETERS END#
```

Test with

```sh
../parse_ham.py --test_file Hamiltonian_Nematic_Dirac_demo_smod.F90
```

Declare additional Hamiltonian-wide variables
```fortran
      Type (Unit_cell), target :: latt_unit_ising

      !> Constanst for the Ising action
      Integer                :: N_ising
      Real (Kind=Kind(0.d0)) :: DW_Ising_tau(-1:1), DW_Ising_Space(-1:1)
      Integer, allocatable   :: Ising_nnlist(:,:)

      !>    Private variables for observing Z_x_ising
      Real (Kind=Kind(0.d0)) :: eq_x_ising, neq_x_ising

      !> Counting measurements
      Integer :: n_measure = 0
```


Populate `Ham_set` body:

- Read parameters

- Set fixed parameters of Main Hamiltonian

- Setup the Bravais lattice

- Setup the hopping / single-particle part

- Setup the interaction.

```fortran
          ! From dynamically generated file "Hamiltonian_Nematic_Dirac_read_write_parameters.F90"
          call read_parameters()

          ! Set fixed parameters of Main Hamiltonian
          N_FL = 1
          Ltrot = nint(beta/dtau)
          Projector = .false.
          Thtrot = 0
          Symm = .false.

          ! Setup the Bravais lattice
          call Ham_Latt

          ! Setup the hopping / single-particle part
          call Ham_Hop

          ! Setup the interaction.
          call Ham_V()
          call Setup_Ising_action
```

- Write parameters to info file

```fortran
             Open(newunit=unit_info, file=file_info, status="unknown", position="append")
             Write(unit_info,*) '====================================='
             Write(unit_info,*) 'Model is            : ', 'Nematic_Dirac'
             Write(unit_info,*) 'L1                  : ', L1
             Write(unit_info,*) 'L2                  : ', L2
             Write(unit_info,*) 'N_SUN               : ', N_SUN
             Write(unit_info,*) 'ham_t               : ', ham_t
             Write(unit_info,*) 'dtau                : ', dtau
             Write(unit_info,*) 'beta                : ', beta
             Write(unit_info,*) 'Ham_h               : ', Ham_h
             Write(unit_info,*) 'Ham_J               : ', Ham_J
             Write(unit_info,*) 'Ham_xi              : ', Ham_xi
             Write(unit_info,*) 'Ham_chem            : ', Ham_chem
             Close(unit_info)
```

### Lattice

Create Lattice

Bravais lattice:
$$\begin{aligned}
\mathbf{a}_1 &= \begin{pmatrix} 1 \\ 1 \end{pmatrix} &
\mathbf{a}_2 &= \begin{pmatrix} 1 \\ -1 \end{pmatrix}
\end{aligned}$$

Unit cell:
$$\begin{aligned}
\text{Fermions:} & \begin{pmatrix} 0 \\ 0 \end{pmatrix}, 
                             \begin{pmatrix} 0 \\ 1 \end{pmatrix} \\
\text{Ising spins:} & \begin{pmatrix} 0 \\ 0 \end{pmatrix}
\end{aligned}$$

![image](latt1_0.pdf)

![image](latt1_Ising.pdf)

```fortran
        subroutine ham_latt()
            implicit none
  
            Real (Kind=Kind(0.d0))  :: a1_p(2), a2_p(2), L1_p(2), L2_p(2)
            Integer :: I, nc, no
  
            latt_unit%Norb = 2
            latt_unit%N_coord = 4
            allocate(latt_unit%orb_pos_p(2, 2))
            latt_unit%orb_pos_p(1, :) = [0.D0,  0.D0]
            latt_unit%orb_pos_p(2, :) = [0.D0,  1.D0]
  
            latt_unit_ising%Norb = 1
            latt_unit_ising%N_coord = 2
            allocate(latt_unit_ising%orb_pos_p(1, 2))
            latt_unit_ising%orb_pos_p(1, :) = [0.D0,  0.D0]
  
            a1_p(:) = [1.D0,  1.D0]
            a2_p(:) = [1.D0, -1.D0]
            L1_p    = dble(L1)*a1_p
            L2_p    = dble(L2)*a2_p
            Call Make_Lattice( L1_p, L2_p, a1_p,  a2_p, Latt)
            If ( L1 == 1 .or. L2 == 1 ) then
              Write(error_unit,*) 'One dimensional systems not implemented'
              error stop
            endif
  
            ! Ndim is needed by the Monte Carlo core!
            Ndim = Latt%N*latt_unit%Norb
  
            ! This is for the orbital structure.
            Allocate (List(Ndim,2), Invlist(Latt%N, latt_unit%Norb))
            nc = 0
            Do I = 1, Latt%N
              Do no = 1, latt_unit%Norb
                nc = nc + 1
                List(nc,1) = I
                List(nc,2) = no
                Invlist(I,no) = nc
              Enddo
            Enddo
          end subroutine ham_latt
```

### Hopping

![image](latt1_0.pdf)

$$\begin{aligned}
  \mathcal{H}_0 & = -t\sum_{\boldsymbol{R}} \sum_{\sigma=1}^{N_\sigma}
                        \hat{a}_{\boldsymbol{R},\sigma}^\dag
                        \Big( \hat{b}_{\boldsymbol{R},\sigma}           \mathrm{e}^{-i\tfrac{\pi}{4}}
                             + \hat{b}_{\boldsymbol{R}+\boldsymbol{e}_-,\sigma}      \mathrm{e}^{ i\tfrac{\pi}{4}} \nonumber \\
        &\quad    + \hat{b}_{\boldsymbol{R}+\boldsymbol{e}_--\boldsymbol{e}_+,\sigma} \mathrm{e}^{-i\tfrac{\pi}{4}}
                             + \hat{b}_{\boldsymbol{R}-\boldsymbol{e}_+,\sigma}      \mathrm{e}^{ i\tfrac{\pi}{4}}
                               \Big) + \mathrm{h.c.},
\end{aligned}$$


- `call Op_make(Op_T(n_check, nf), size_of_op)`

- Fill operator

- `call Op_set(Op_T(n_check, nf))`

Sum over $\sigma$ is builtin.

```fortran
        subroutine ham_hop()
            implicit none
  
            Integer :: I, I1, J1, nc1
            complex (Kind=Kind(0.d0)) :: t_temp
  
            allocate(Op_T(1, N_FL))
            Call Op_make(Op_T(1, 1), Ndim)
  
            DO I = 1, Latt%N
              I1 = Invlist(I,1)
              Do nc1 = 1, 4
                select case (nc1)
                case (1)
                  J1 = invlist(Latt%nnlist(I, 0, 0), 2)
                  t_temp = -Ham_T * cmplx(1, -1, kind(0.D0))/sqrt(2.D0)
                case (2)
                  J1 = invlist(Latt%nnlist(I, 0, 1), 2)
                  t_temp = -Ham_T * cmplx(1,  1, kind(0.D0))/sqrt(2.D0)
                case (3)
                  J1 = invlist(Latt%nnlist(I, -1, 1), 2)
                  t_temp = -Ham_T * cmplx(1, -1, kind(0.D0))/sqrt(2.D0)
                case (4)
                  J1 = invlist(Latt%nnlist(I, -1, 0), 2)
                  t_temp = -Ham_T * cmplx(1,  1, kind(0.D0))/sqrt(2.D0)
                end select
                Op_T(1, 1)%O(I1,J1) = t_temp
                Op_T(1, 1)%O(J1,I1) = conjg(t_temp)
              Enddo
            Enddo
  
            Do I = 1,Ndim
               Op_T(1,1)%P(i) = i
               Op_T(1,1)%O(i,i) = cmplx(-Ham_chem, 0.d0, kind(0.D0))
            Enddo
            Op_T(1,1)%g = -Dtau
            Op_T(1,1)%alpha=cmplx(0.d0,0.d0, kind(0.D0))
            Call Op_set(Op_T(1,1))
          end subroutine ham_hop
```

### Interaction

![image](latt1_Yuk.pdf)
$$\begin{aligned}
  \mathcal{H}_\text{Yuk} & = -\xi \sum_{\boldsymbol{R}} \sum_{\sigma=1}^{N_\sigma}
                 \hat{s}_{\boldsymbol{R}}^z \hat{a}^\dag_{\boldsymbol{R},\sigma}
                 \Big( \hat{b}_{\boldsymbol{R},\sigma}           \mathrm{e}^{-i\tfrac{\pi}{4}}
                      - \hat{b}_{\boldsymbol{R}+\boldsymbol{e}_-,\sigma}      \mathrm{e}^{ i\tfrac{\pi}{4}}
                      \nonumber\\
   &\quad         - \hat{b}_{\boldsymbol{R}+\boldsymbol{e}_--\boldsymbol{e}_+,\sigma} \mathrm{e}^{-i\tfrac{\pi}{4}}
                      + \hat{b}_{\boldsymbol{R}-\boldsymbol{e}_+,\sigma}      \mathrm{e}^{ i\tfrac{\pi}{4}}
                               \Big) + \mathrm{h.c.},
\end{aligned}$$

- `call Op_make(Op_V(n_int, nf), size_of_op)`

- Fill operator

- `call Op_set( Op_V(n_int, nf) )`

Sum over $\sigma$ is builtin.

```fortran
        subroutine Ham_V()
            implicit none
            integer :: I, J1, nc1
            complex (Kind=Kind(0.d0)) :: t_temp
  
            Allocate(Op_V(Latt%N, N_FL))
            do I = 1, Latt%N
              call Op_make(Op_V(I, 1), 5)
              Op_V(I, 1)%P(1) = Invlist(I,1)
              Do nc1 = 1, 4
                select case (nc1)
                case (1)
                  J1 = invlist(Latt%nnlist(I, 0, 0), 2)
                  t_temp = -cmplx(1, -1, kind(0.D0))/sqrt(2.D0)
                case (2)
                  J1 = invlist(Latt%nnlist(I, 0, 1), 2)
                  t_temp =  cmplx(1,  1, kind(0.D0))/sqrt(2.D0)
                case (3)
                  J1 = invlist(Latt%nnlist(I, -1, 1), 2)
                  t_temp =  cmplx(1, -1, kind(0.D0))/sqrt(2.D0)
                case (4)
                  J1 = invlist(Latt%nnlist(I, -1, 0), 2)
                  t_temp = -cmplx(1,  1, kind(0.D0))/sqrt(2.D0)
                end select
                Op_V(I, 1)%P(nc1+1) = J1
                Op_V(I, 1)%O(1   ,nc1+1) = t_temp
                Op_V(I, 1)%O(nc1+1,1   ) = conjg(t_temp)
              Enddo
              Op_V(I, 1)%g     = cmplx(-dtau*ham_xi,0.d0, kind(0.D0))
              Op_V(I, 1)%alpha = cmplx(0.d0,0.d0, kind(0.D0))
              Op_V(I, 1)%type  = 1  ! Set to Ising type interaction
              Call Op_set( Op_V(I, 1) )
            Enddo
  
          end subroutine Ham_V
```


Ising action: Mapping to $d$+1-dimensional classical Ising model
$$\begin{aligned}
  \mathcal{H}_\text{Ising} = -J \sum_{\langle \boldsymbol{R},\boldsymbol{R}' \rangle} \hat{s}_{\boldsymbol{R}}^z \hat{s}_{\boldsymbol{R}'}^z -h \sum_{\boldsymbol{R}} \hat{s}_{\boldsymbol{R}}^x,
\end{aligned}$$

Trotter decomposition:
$$
Z &= \text{Tr}\left[\exp(-\beta H_\text{Ising})\right] \\
Z &= \sum_\Phi \Braket{\Phi | \left(
       \exp\left(\Delta_{\tau} J \sum_{\langle \boldsymbol{R}, \boldsymbol{R}' \rangle} \hat{s}^z_{\boldsymbol{R}} \hat{s}^z_{\boldsymbol{R}'} \right)
       \exp\left(\Delta_\tau h\sum_{\boldsymbol{R}} \hat{s}^x_{\boldsymbol{R}} \right)
         \right)
       ^{L_\tau} |\Phi} + \mathcal{O}({\Delta_\tau}^2)
$$

State: $d$ $\rightarrow$ $d+1$ dimensions
$$
Z &= \sum_{C} \prod_{t=1}^{L_\tau}
       \left[ \exp\left(\Delta_\tau J \sum_{\boldsymbol{R},\boldsymbol{R}} s_{\boldsymbol{R},t}^z s_{\boldsymbol{R}',t}^z \right)
            \Braket{\Phi_t | \exp\left(h \Delta_\tau \sum_{\boldsymbol{R}} \hat{s}^x_{\boldsymbol{R}} \right) | \Phi_{t+1} } \right]
$$


Ising action
$$\begin{aligned}
&Z = \lambda^{N L_\tau}
      \sum_{C} e^{-S_{\text{Ising}}(C)} + \mathcal{O}({\Delta_\tau}^2)\\
&e^{-S_{\text{Ising}}(C)} = 
      \prod_{t=1}^{L_\tau}
       \left[ \exp\left(\Delta_\tau J \sum_{\boldsymbol{R},\boldsymbol{R}'} s_{\boldsymbol{R},t}^z s_{\boldsymbol{R}',t}^z \right)
              \exp\left( \gamma\sum_{\boldsymbol{R}} s^z_{\boldsymbol{R},t} s^z_{\boldsymbol{R},t+1} \right)
       \right]
       \\
       &\qquad \gamma = -\tfrac{1}{2}\ln( \tanh\, h\Delta_\tau)
       \qquad \lambda = \sqrt{\sinh(h\Delta_\tau)\cosh(h\Delta_\tau)}
\end{aligned}$$
Single spin flip:
$$\begin{aligned}
\frac{e^{-S_{\text{Ising}}(C_\text{new})}} {e^{-S_{\text{Ising}}(C_\text{old})}} = \left\{
\begin{array}{ll}
\tanh(\Delta_\tau h)   & s^z_{\boldsymbol{R},t} s^z_{\boldsymbol{R},t+1}: 1 \rightarrow -1
\\
1/\tanh(\Delta_\tau h) & s^z_{\boldsymbol{R},t} s^z_{\boldsymbol{R},t+1}: -1 \rightarrow 1
\\
\exp(-2 \Delta_\tau J) & s_{\boldsymbol{R},t}^z s_{\boldsymbol{R}',t}^z: 1 \rightarrow -1
\\
\exp(2 \Delta_\tau J) & s_{\boldsymbol{R},t}^z s_{\boldsymbol{R}',t}^z: -1 \rightarrow 1
\end{array}
\right.
\end{aligned}$$

```fortran
        Subroutine Setup_Ising_action()

            ! This subroutine sets up lists and arrays to enable an
            ! an efficient calculation of  S0(n,nt)
  
            Implicit none
            Integer :: I
  
            allocate(Ising_nnlist(Latt%N,4))
            N_ising = Latt%N
            do I = 1, Latt%N
              Ising_nnlist(I,1) = Latt%nnlist(I, 1, 0)
              Ising_nnlist(I,2) = Latt%nnlist(I, 0, 1)
              Ising_nnlist(I,3) = Latt%nnlist(I,-1, 0)
              Ising_nnlist(I,4) = Latt%nnlist(I, 0,-1)
            enddo
  
            ! exp(-S0(new))/exp(-S0(old)) of one Ising bond. Index is sa*sb
            DW_Ising_tau  ( 1) = tanh(Dtau*Ham_h)
            DW_Ising_tau  (-1) = 1.D0/DW_Ising_tau(1)
            DW_Ising_Space( 1) = exp(-2.d0*Dtau*Ham_J)
            DW_Ising_Space(-1) = exp( 2.d0*Dtau*Ham_J)
          End Subroutine Setup_Ising_action

! Define Ising dynamics

!===================================================================================
!--------------------------------------------------------------------
!> @author
!> ALF Collaboration
!>
!> @brief
!> Single spin flip S0 ratio
!> @details
!> S0=exp(-S0(new))/exp(-S0(old)) where the new configuration correpsonds to the old one up to
!> a spin flip of Operator n on time slice nt
!> @details
!--------------------------------------------------------------------
          Real (Kind=Kind(0.d0)) function S0(n,nt,Hs_new)
          Implicit none
          !> Operator index
          Integer, Intent(IN) :: n
          !> Time slice
          Integer, Intent(IN) :: nt
          !> New local field on time slice nt and operator index n
          Real (Kind=Kind(0.d0)), Intent(In) :: Hs_new

          Integer :: nt1,I
          S0 = 1.d0
          If ( Op_V(n,1)%type == 1 ) then
              do i = 1,4
                S0 = S0*DW_Ising_space(nsigma%i(n,nt)*nsigma%i(Ising_nnlist(n,i),nt))
              enddo
              nt1 = nt +1
              if (nt1 > Ltrot) nt1 = 1
              S0 = S0*DW_Ising_tau(nsigma%i(n,nt)*nsigma%i(n,nt1))
              nt1 = nt - 1
              if (nt1 < 1  ) nt1 = Ltrot
              S0 = S0*DW_Ising_tau(nsigma%i(n,nt)*nsigma%i(n,nt1))
              If (S0 < 0.d0) Write(6,*) 'S0 : ', S0
          endif

        end function S0


!--------------------------------------------------------------------
!> @author
!> ALF Collaboration
!>
!> @brief
!> Computes the ratio exp(S0(new))/exp(S0(old))
!>
!> @details
!> This function computes the ratio \verbatim  e^{-S0(nsigma)}/e^{-S0(nsigma_old)} \endverbatim
!> @param [IN] nsigma_old,  Type(Fields)
!> \verbatim
!>  Old configuration. The new configuration is stored in nsigma.
!> \endverbatim
!-------------------------------------------------------------------
        Real (Kind=kind(0.d0)) Function Delta_S0_global(nsigma_old)
      
          !>  This function computes the ratio:  e^{-S0(nsigma%f)}/e^{-S0(nsigma_old)}
          Implicit none
      
          !> Arguments
          type(fields), intent(in) :: nsigma_old
      
          !> Local
          Integer :: I, nt, nt1, I1, I2, nc_J, nc_h_p, nc_h_m
      
          Delta_S0_global = 1.D0
          nc_J = 0    ! Number of parallel minus number of anti-parallel Ising bonds in space
          nc_h_p = 0  ! Number of parallel Ising bonds in time
          nc_h_m = 0  ! Number of anti-parallel Ising bonds in time
      
          Do I = 1, latt%N
            I1 = latt%nnlist(I,1,0)
            I2 = latt%nnlist(I,0,1)
            Do nt = 1,Ltrot
                nt1 = nt + 1
                if (nt == Ltrot) nt1 = 1
                if (nsigma%i(I,nt) == nsigma%i(I,nt1) ) then
                  nc_h_p = nc_h_p + 1
                else
                  nc_h_m = nc_h_m + 1
                endif
                if (nsigma_old%i(I,nt) == nsigma_old%i(I,nt1) ) then
                  nc_h_p = nc_h_p - 1
                else
                  nc_h_m = nc_h_m - 1
                endif
      
                nc_J = nc_J + nsigma%i(I,nt)*nsigma%i(I1,nt) &
                    &      + nsigma%i(I,nt)*nsigma%i(I2,nt) &
                    &      - nsigma_old%i(I,nt)*nsigma_old%i(I1,nt) &
                    &      - nsigma_old%i(I,nt)*nsigma_old%i(I2,nt)
            enddo
          enddo
      
          Delta_S0_global = ( sinh(Dtau*Ham_h)**nc_h_m ) * (cosh(Dtau*Ham_h)**nc_h_p) &
                  &         * exp( Dtau * Ham_J*real(nc_J,kind(0.d0)))
      
        end Function Delta_S0_global
```

## Define Observables

### Alloc_obs

Create Observables: `Alloc_obs` Allocate the storage space for
Observables

- `Part`: Fermion particle number

- `ising_z`: $\langle \hat{s}^z \rangle$

- `Kin_Pot_E`: $\mathcal{H}_0$, $\mathcal{H}_\text{Yuk}$,
  $\mathcal{H}_0 + \mathcal{H}_\text{Yuk}$

- `ising_x`:
  $\langle \hat{s}^x \rangle = \langle \partial_h \mathcal{H} \rangle$

- `m`:
  $\sum_{\boldsymbol{R}} \langle \hat{s}^z_{\boldsymbol{0}} \hat{s}^z_{\boldsymbol{R}} \rangle$
  and its 2nd, 4th moment

- `IsingX`:
  $\sum_{\boldsymbol{R}'} \langle \hat{s}^x_{\boldsymbol{R}'} \hat{s}^x_{\boldsymbol{R}'+\boldsymbol{R}} \rangle$

- `Green`:
  $\sum_{\boldsymbol{R}'} \langle \hat{c}^\dag_{\boldsymbol{R}'} \hat{c}_{\boldsymbol{R}'+\boldsymbol{R}} \rangle$

- `Den`:
  $\sum_{\boldsymbol{R}'} \langle \hat{n}_{\boldsymbol{R}'} \hat{n}_{\boldsymbol{R}'+\boldsymbol{R}} \rangle$

```fortran
          ! Scalar observables
          Allocate ( Obs_scal(5) )
          Do I = 1,Size(Obs_scal,1)
            select case (I)
            case (1)
              N = 1;   Filename ="Part"
            case (2)
              N = 1;   Filename ="ising_z"
            case (3)
              N = 3;   Filename ="Kin_Pot_E"
            case (4)
              N = 1;   Filename ="ising_x"
              eq_x_ising  = tanh(Dtau*Ham_h)
              neq_x_ising = 1/tanh(Dtau*Ham_h)
            case (5)
              N = 3;   Filename ="m"
            case default
              Write(6,*) ' Error in Alloc_obs scal'
            end select
            call Obser_Vec_make(Obs_scal(I), N, Filename)
          enddo
        
        
          ! Equal time correlators
          Allocate ( Obs_eq(6) )
          Channel = '--'
          Do I = 1,Size(Obs_eq,1)
            select case (I)
            case (1)
              Filename ="IsingX"
              Nt = 1
              call Obser_Latt_make(Obs_eq(I), Nt, Filename, Latt, Latt_unit_ising, Channel, dtau)
            case (2)
              Filename ="IsingZ"
              Nt = 1
              call Obser_Latt_make(Obs_eq(I), Nt, Filename, Latt, Latt_unit_ising, Channel, dtau)
            case (3)
              Filename ="Green"
              Nt = 1
              call Obser_Latt_make(Obs_eq(I), Nt, Filename, Latt, Latt_unit, Channel, dtau)
            case (4)
              Filename ="Den"
              Nt = 1
              call Obser_Latt_make(Obs_eq(I), Nt, Filename, Latt, Latt_unit, Channel, dtau)
            case (5)
              Filename ="IsingZT"
              Nt = Ltrot+1
              call Obser_Latt_make(Obs_eq(I), Nt, Filename, Latt, Latt_unit_ising, Channel, dtau)
            case (6)
              Filename ="IsingXT"
              Nt = Ltrot+1
              call Obser_Latt_make(Obs_eq(I), Nt, Filename, Latt, Latt_unit_ising, Channel, dtau)
            case default
              Write(6,*) ' Error in Alloc_obs eq'
            end select
          enddo
        
          If (Ltau == 1) then
             ! Equal time correlators
             Allocate ( Obs_tau(2) )
             Do I = 1,Size(Obs_tau,1)
                select case (I)
                case (1)
                  Filename ="Green"
                case (2)
                  Filename ="Den"
                case default
                   Write(6,*) ' Error in Alloc_obs tau'
                end select
                Nt = Ltrot+1
                Channel = '--'
                call Obser_Latt_make(Obs_tau(I), Nt, Filename, Latt, Latt_unit, Channel, dtau)
             enddo
          endif
```

Generic funtion for kinetic energy

```fortran
        function E_kin(GRC)
            Implicit none
            Complex (Kind=Kind(0.d0)), intent(in) :: GRC(Ndim,Ndim,N_FL)
         
            Complex (Kind=Kind(0.d0)) :: E_kin
         
            Integer :: n, nf, I, J
         
            E_kin = cmplx(0.d0, 0.d0, kind(0.D0))
            Do n  = 1,Size(Op_T,1)
               Do nf = 1,N_FL
                  Do I = 1,Size(Op_T(n,nf)%O,1)
                     Do J = 1,Size(Op_T(n,nf)%O,2)
                        E_kin = E_kin + Op_T(n,nf)%O(i, j)*Grc( Op_T(n,nf)%P(I), Op_T(n,nf)%P(J), nf )
                     ENddo
                  Enddo
               Enddo
            Enddo
            E_kin = E_kin * dble(N_SUN)
         end function E_kin
```

### Obser

Calculate equal time observables: `Obser`

Scalar observables

- `Obs_scal(I)%N`: Number of measurements

- `Obs_scal(I)%Ave_sign`: Average sign

- `Obs_scal(I)%Obs_vec(n)`: Measured value

Correlation functions

- `Obs_scal(I)%N`: Number of measurements

- `Obs_scal(I)%Ave_sign`: Average sign

- `Obs_scal(I)%Obs_Latt0(no)`:\
  Background $\Braket{\hat{O}_{\boldsymbol{R}, no}}$

- `Obs_scal(I)%Obs_Latt(imj, nt, no1, no2)`:\
  Correlation
  $\Braket{\hat{O}_{\boldsymbol{R}, no1}\hat{O}_{\boldsymbol{R}+\boldsymbol{R}(imj), no2}}$

```fortran
          Complex (Kind=Kind(0.d0)) :: Z_z_ising, Z_x_ising, Z_m
          Complex (Kind=Kind(0.d0)) :: Zkin, Zpot, Zrho
          Integer :: nc1, imj, nt1, nt, dnt, Ntau1, n, I1



          ! Compute scalar observables.
          Do I = 1,Size(Obs_scal,1)
            Obs_scal(I)%N         =  Obs_scal(I)%N + 1
            Obs_scal(I)%Ave_sign  =  Obs_scal(I)%Ave_sign + Real(ZS,kind(0.d0))
         Enddo
  
          Z_z_ising = cmplx(0.d0, 0.d0, kind(0.D0))
          Z_x_ising = cmplx(0.d0, 0.d0, kind(0.D0))
          Z_m       = cmplx(0.d0, 0.d0, kind(0.D0))

          nt1 = Ntau+1; if ( Ntau == Ltrot ) nt1 = 1

          ! Particle number
          Zrho = cmplx(0.d0, 0.d0, kind(0.D0))
          do I = 1, Ndim
            Zrho = Zrho + Grc(i, i, 1)
          enddo
          Zrho = Zrho * dble(N_SUN)
          Obs_scal(1)%Obs_vec(1) = Obs_scal(1)%Obs_vec(1) + Zrho *ZP*ZS
        
          ! Kinetic energy
          Zkin = E_kin(GRC)
        
          ! Potential energy
          Zpot = cmplx(0.d0, 0.d0, kind(0.D0))
          do I = 1, size(Op_V, 1)
            I1 = Op_V(I,1)%P(1)
            do nc1 = 2, size( Op_V(I,1)%O, 1 )
              Zpot = Zpot + nsigma%f(I,Ntau) * GRC(I1, Op_V(I,1)%P(nc1) ,1) * Op_V(I,1)%O(1,nc1)
              Zpot = Zpot + nsigma%f(I,Ntau) * GRC(Op_V(I,1)%P(nc1), I1 ,1) * Op_V(I,1)%O(nc1,1)
            enddo
          enddo
          Zpot = Zpot * dble(N_SUN) * ham_t * ham_xi
          Obs_scal(3)%Obs_vec(1) = Obs_scal(3)%Obs_vec(1) + Zkin *ZP*ZS
          Obs_scal(3)%Obs_vec(2) = Obs_scal(3)%Obs_vec(2) + Zpot *ZP*ZS
          Obs_scal(3)%Obs_vec(3) = Obs_scal(3)%Obs_vec(3) + Zkin+Zpot *ZP*ZS
        
        
          do I = 1,Latt%N
            Z_z_ising = Z_z_ising + nsigma%f(I,Ntau)
            Z_m = Z_m + nsigma%f(1,Ntau)*nsigma%f(I,Ntau)
        
            if ( nsigma%f(I,nt1) == nsigma%f(I,Ntau) ) then
              Z_x_ising = Z_x_ising + eq_x_ising
            else
              Z_x_ising = Z_x_ising + neq_x_ising
            endif
          enddo
          Z_z_ising = Z_z_ising/Latt%N
          Z_x_ising = Z_x_ising/Latt%N
          Z_m = Z_m/Latt%N

          Obs_scal(2)%Obs_vec(1) = Obs_scal(2)%Obs_vec(1) + Z_z_ising *ZP*ZS
          Obs_scal(4)%Obs_vec(1) = Obs_scal(4)%Obs_vec(1) + Z_x_ising *ZP*ZS
          Obs_scal(5)%Obs_vec(1) = Obs_scal(5)%Obs_vec(1) + Z_m *ZP*ZS
          Obs_scal(5)%Obs_vec(2) = Obs_scal(5)%Obs_vec(2) + Z_m**2 *ZP*ZS
          Obs_scal(5)%Obs_vec(3) = Obs_scal(5)%Obs_vec(3) + Z_m**4 *ZP*ZS
      
          ! Compute equal-time correlations
      
          ! counting up correlation functions
          ! DO I = 1,Size(Obs_eq,1)
          DO I = 1,2
            Obs_eq(I)%N        = Obs_eq(I)%N + 1
            Obs_eq(I)%Ave_sign = Obs_eq(I)%Ave_sign + real(ZS,kind(0.d0))
          ENDDO
        
          ! Compute Ising X-X and Z-Z correlation functions
          nt1 = Ntau+1; if ( Ntau == Ltrot ) nt1 = 1
          Do I = 1,Latt%N
            if ( nsigma%f(I,nt1) == nsigma%f(I,Ntau) ) then
              Z_x_ising = eq_x_ising
            else
              Z_x_ising = neq_x_ising
            endif
            Obs_eq(1)%Obs_Latt0(1) = Obs_eq(1)%Obs_Latt0(1) + Z_x_ising * ZP*ZS
            ! Improved estimator: <s_z> = 0 
            ! Obs_eq(2)%Obs_Latt0(1) = Obs_eq(2)%Obs_Latt0(1) + nsigma%f(I,Ntau) * ZP*ZS
            Do J = 1,Latt%N
              imj = Latt%imj(I,J)
              Obs_eq(2)%Obs_Latt(imj,1,1,1) = Obs_eq(2)%Obs_Latt(imj,1,1,1) + nsigma%f(I,Ntau) * nsigma%f(J,Ntau) * ZP*ZS
              if ( nsigma%f(J,nt1) == nsigma%f(J,Ntau) ) then
                Obs_eq(1)%Obs_Latt(imj,1,1,1) = Obs_eq(1)%Obs_Latt(imj,1,1,1) + Z_x_ising * eq_x_ising  * ZP*ZS
              else
                Obs_eq(1)%Obs_Latt(imj,1,1,1) = Obs_eq(1)%Obs_Latt(imj,1,1,1) + Z_x_ising * neq_x_ising * ZP*ZS
              endif
            enddo
          enddo
 
          ! Compute Green-function
          call Predefined_Obs_eq_Green_measure( Latt, Latt_unit, List,  GR, GRC, N_SUN, ZS, ZP, Obs_eq(3) )

          ! Compute Density-Density correlations
          call Predefined_Obs_eq_Den_measure( Latt, Latt_unit, List,  GR, GRC, N_SUN, ZS, ZP, Obs_eq(4) )
       
         ! Computing time-displaced X-X and Z-Z correlation functions
          n_measure = n_measure + 1
          if ( n_measure == Ltrot ) then
            n_measure = 0
        
            DO I = 5,6
              Obs_eq(I)%N        = Obs_eq(I)%N + 1
              Obs_eq(I)%Ave_sign = Obs_eq(I)%Ave_sign + real(ZS,kind(0.d0))
            ENDDO
          
            Z_x_ising = cmplx(0.d0, 0.d0, kind(0.D0))
            Ntau1 = Ntau+1; if ( Ntau == Ltrot ) Ntau1 = 1
          
            do dnt= 0, Ltrot
              nt = Ntau + dnt
              if ( nt > Ltrot ) nt = nt - Ltrot
              nt1 = nt+1; if ( nt == Ltrot ) nt1 = 1
              do I = 1,Latt%N
                if ( nsigma%f(I,nt1) == nsigma%f(I,nt) ) then
                  Z_x_ising = eq_x_ising
                else
                  Z_x_ising = neq_x_ising
                endif
                ! Improved estimator: <s_z> = 0 
                ! Obs_eq(5)%Obs_Latt0(1) = Obs_eq(5)%Obs_Latt0(1) + nsigma%f(I,nt) * ZP*ZS
                Obs_eq(6)%Obs_Latt0(1) = Obs_eq(6)%Obs_Latt0(1) + Z_x_ising    * ZP*ZS
                do J = 1,Latt%N
                  imj = Latt%imj(I,J)
                  Obs_eq(5)%Obs_Latt(imj,dnt+1,1,1) = Obs_eq(5)%Obs_Latt(imj,dnt+1,1,1) + nsigma%f(I,Ntau) * nsigma%f(J,nt) * ZP*ZS
                  if ( nt == Ntau .and. I == J ) then
                    Obs_eq(6)%Obs_Latt(imj,dnt+1,1,1) = Obs_eq(6)%Obs_Latt(imj,dnt+1,1,1) + 1  * ZP*ZS
                  elseif ( nsigma%f(J,Ntau1) == nsigma%f(J,Ntau) ) then
                    Obs_eq(6)%Obs_Latt(imj,dnt+1,1,1) = Obs_eq(6)%Obs_Latt(imj,dnt+1,1,1) + Z_x_ising * eq_x_ising  * ZP*ZS
                  else
                    Obs_eq(6)%Obs_Latt(imj,dnt+1,1,1) = Obs_eq(6)%Obs_Latt(imj,dnt+1,1,1) + Z_x_ising * neq_x_ising * ZP*ZS
                  endif
                enddo
              enddo
            enddo
       
          endif
```

### ObserT

Calculate time-displaced observables: `ObserT` Analogous to equal time
correlation functions

```fortran
          call Predefined_Obs_tau_Green_measure( Latt, Latt_unit, List, NT, GT0,G0T,G00,GTT,  N_SUN, ZS, ZP, Obs_tau(1) )
          call Predefined_Obs_tau_Den_measure( Latt, Latt_unit, List, NT, GT0,G0T,G00,GTT,  N_SUN, ZS, ZP, Obs_tau(2) )
```


## Finishing

- Add `Nematic_Dirac_demo` to `Prog/Hamiltonians.list`
- (Try to) compile
- Test run
