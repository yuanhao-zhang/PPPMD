# PPPMD
Post simulation analysis
import numpy as np
antype = 'TFSI'
freefrac = 0.25


N_polymer =10  #PMLiTFSI
N_homo = 0 #POEM

L_homo = 6
LA = 50
LB = 0
L_polymer = LA + LB
F_A = 1

EOLi = 10

L_branch_poly = 3
L_branch_homo = 11 # 11

Branchinterval_poly = 0
Branchinterval_homo = 0
###################################### atom type
bcpbackboneAty = 2 # if block polymer, that's the type of block with branches
bcpbackboneBty = 2 # if block polymer, that's the type of linear block
bcpbranchty = 1 # type of branch on PLiMTFSI

tfsi_atom_a_type = 7 #TF group in TFSI
tfsi_atom_b_type = 8 #N  group in TFSI

tf_atom_a_type = 7
tf_atom_b_type = 8

pec_atom_type = 7

homobackbonety = 2 #bb of POEM
homobranchty = 1   #branch of POEM 
ethyenety = 6
cation_ty = 3
anionAty = 4       #TF group in TFSI
anionBty = 5       #N  group in TFSI
###################################### bond type
bcpbackboneAbty = 2 # if block polymer, that's the type of block with branches
bcpbackboneBbty = 2 # if block polymer, that's the type of linear block

homo_eo_bty = 1   # PEO bond type in POEM
anionbty = 3      # TF- N bond of free ions
abtyp = 5         # EO - TF bond of branch ions            PMMA - EO - N - TF 
homo_eo_mmaty = 4 # PMMA - EO bond 

#################################### angle type 
bcpbackboneBaty = 2#2
bcpbackboneAaty = 2
bcpbranchaty = 1
bcpbranchaty_singleion = 1
eommaeoaty=3
mmammaeoaty=4
eommaeo=True
mmammaeo = True

#####################################
bonda = 1.0
bondb = 2.0
bondc = 1.0
density = 0.3
N_rep = 2

if L_branch_poly == 0:
    N_rep =0
if EOLi == 0:
    ethyenety = 1
    bcpbranchaty_singleion = 1

n_anion = N_rep * N_polymer * L_polymer
#n_anion = 0
freeions = int(freefrac*n_anion)
#freeions = int(800)
print(float(freeions)/float(freeions+n_anion))
n_EO = (n_anion + freeions)*EOLi
EOperchain = N_rep*L_branch_homo*L_homo
N_homo = int(n_EO/EOperchain)
realratio = float(N_homo*EOperchain)/float((freeions+n_anion))
if antype =='TFSI':
    anioncount = 3
    mass=[0, 1.0, 5.7, 0.2, 2.0, 4.1, 1.0, 2.0, 4.1]
if antype =='TF':
    anioncount = 2
    mass=[0, 1.0, 5.7, 0.2, 2.0, 4.1, 1.0, 2.0, 4.1]
if antype =='PEC':
    anioncount = 1
    mass=[0, 1.0, 5.7, 0.2, 2.0, 4.1, 1.0, 2.0, 4.1]


#N_homo = 0
N_branch_poly = N_polymer * int(LA / (Branchinterval_poly + 1)) * L_branch_poly * N_rep
N_branch_homo = N_homo * int(L_homo / (Branchinterval_homo + 1)) * L_branch_homo * N_rep
N_total = int(N_branch_homo + N_branch_poly + N_polymer * L_polymer + N_homo * L_homo  + n_anion + freeions*(anioncount+1))#the branch is also anion, so here only count the Li
volume = N_total / density
overall = int(N_total)+1

L = volume**(1.0/3.0)

resid = 0
tmpcount_bcp=int(L_polymer + (LA / (Branchinterval_poly + 1)) * L_branch_poly*N_rep)+1
tmpcount_hp=int(L_homo + (L_homo / (Branchinterval_homo + 1)) * L_branch_homo*N_rep)+1
cbcp = np.zeros((N_polymer+1,tmpcount_bcp, 3))
chp = np.zeros((N_homo+1,tmpcount_hp, 3))

alength = np.zeros(N_polymer)
index = 0

def randomwalk(bond):
    theta = np.random.rand() * 2 * np.pi
    dz = 2 * np.random.rand() - 1
    dx = np.sqrt(1 - dz**2) * np.cos(theta)
    dy = np.sqrt(1 - dz**2) * np.sin(theta)
    r = np.sqrt(dx**2 + dy**2 + dz**2)
    scale = bond / r
    dx *= scale
    dy *= scale
    dz *= scale
    return dx, dy, dz


# construct BCP
for i in range(1,N_polymer+1):             #constract backbone of BCP
    ind_branch = 0
    for j in range(1,LB+1):                #construct B block 
        dx, dy, dz = randomwalk(bondb)
        tmpind = LA + j
        if j == 1:
            cbcp[i][tmpind, :] = np.array([0, 0, 0])
        if j > 1:
            cbcp[i][tmpind, :] = cbcp=[i][tmpind - 1, :] + np.array([dx, dy, dz])

    for j in range(1,LA+1):                #construct A block
        dx, dy, dz = randomwalk(bondb)
        tmpind = LA - (j - 1)
        if j == 1:
            cbcp[i][tmpind, :] = [dx, dy, dz]
        if j > 1:
            cbcp[i][tmpind, :] = cbcp[i][tmpind + 1, :] + [dx, dy, dz]

               
    for j1 in range(1,LA+1):                #construct branches on A block
        tmp = 1 + (j1 - 1) * (1 + Branchinterval_poly)# determine the position of grafted point on backbone
        for k in range(1,N_rep+1):
            for j2 in range(1,L_branch_poly+1):
                dx, dy, dz = randomwalk(bonda)
                ind_branch += 1
                if j2 == 1:
                    cbcp[i][LA + LB + ind_branch, :] = cbcp[i][tmp, :] + [dx, dy, dz]
                if j2 > 1:
                    cbcp[i][LA + LB + ind_branch, :] = cbcp[i][LA+LB+ind_branch-1, :] + [dx, dy, dz]

# construct HP
for i in range(1,N_homo+1):             #constract backbone of BCP
    ind_branch = 0
    for j in range(1,L_homo+1):                #construct homopolymer backbone
        dx, dy, dz = randomwalk(bondb)
        tmpind = L_homo - (j - 1)
        if j == 1:
            chp[i][tmpind, :] = [dx, dy, dz]
        if j > 1:
            chp[i][tmpind, :] = chp[i][tmpind + 1, :] + [dx, dy, dz]             
    for j1 in range(1,L_homo+1):                #construct branches on A block
        tmp = 1 + (j1 - 1) * (1 + Branchinterval_homo)# determine the position of grafted point on backbone
        for k in range(1,N_rep+1):
            for j2 in range(1,L_branch_homo+1):
                dx, dy, dz = randomwalk(bonda)
                ind_branch += 1
                if j2 == 1:
                    chp[i][L_homo + ind_branch, :] = chp[i][tmp, :] + [dx, dy, dz]
                if j2 > 1:
                    chp[i][L_homo + ind_branch, :] = chp[i][L_homo+ind_branch-1, :] + [dx, dy, dz]


index = 0
# asign the position of polymer
for j in range(1,N_polymer+1):
    index += 1
    zplane = np.random.rand() * L
    xplane = np.random.rand() * L
    yplane = np.random.rand() * L
    tmp=cbcp[j]     
    cbcp[j][:,0]+=[i+xplane for i in np.zeros(len(tmp[:,1]))]
    cbcp[j][:,1]+=[i+yplane for i in np.zeros(len(tmp[:,1]))]
    cbcp[j][:,2]+=[i+zplane for i in np.zeros(len(tmp[:,1]))]

for j in range(1,N_homo+1):
    index += 1
    zplane = np.random.rand() * L
    xplane = np.random.rand() * L
    yplane = np.random.rand() * L
    tmp=chp[j]     
    chp[j][:,0]+=[i+xplane for i in np.zeros(len(tmp[:,1]))]
    chp[j][:,1]+=[i+yplane for i in np.zeros(len(tmp[:,1]))]
    chp[j][:,2]+=[i+zplane for i in np.zeros(len(tmp[:,1]))]



index = 0
overall=int(overall)
atomid = np.zeros(overall)
chainid = np.zeros(overall)
charge = np.zeros(overall)
typeid = np.zeros(overall)
position = np.zeros((overall, 3))
imagenumber = np.zeros((overall, 3), dtype = int)
anionposition = np.zeros((n_anion*3+1, 3))

indbb=np.zeros((N_polymer*L_polymer+N_homo*L_homo+1))
indbr1=np.zeros((N_polymer*L_polymer+N_homo*L_homo+1))
indbr2=np.zeros((N_polymer*L_polymer+N_homo*L_homo+1))

#generate the info of PLiMTFSI
for i in range(1,N_polymer+1):
    cbcptmp = cbcp[i]
    for j in range(1,L_polymer+1):
        index = index + 1
        atomid[index] = index
        chainid[index] = i
        charge[index] = 0
        if j <= LA:
            typeid[index] = bcpbackboneAty
        else:
            typeid[index] = bcpbackboneBty
        position[index, :] = cbcptmp[j, :]
        imagenumber[index, :] = [0, 0, 0]
        indbb[index]=index
indexbbcount = index

index_1_backbone_BCP=index
indexb = index
co1 = 0
co2 = 0
N_anion=0
for i in range(1,N_polymer+1):
    cbcptmp = cbcp[i]
    tmp = 0
    for k1 in range(1,int(LA / (Branchinterval_poly + 1))+1):
        for k in range(1,N_rep+1):
            for j in range(1,L_branch_poly+1):
                tmp = tmp + 1
                index = index + 1
                atomid[index] = index
                chainid[index] = i
                charge[index] = 0
                typeid[index] = ethyenety
                position[index, :] = cbcptmp[L_polymer + tmp, :]
                imagenumber[index, :] = [0, 0, 0]
                if j == 2:
                    charge[index] = -0.85
                    typeid[index] = anionBty
                    N_anion = N_anion + 1
                    anionposition[N_anion, :] = position[index, :]
                if j == 3:
                    charge[index] = -0.15
                    typeid[index] = anionAty
                    N_anion = N_anion + 1
                    anionposition[N_anion, :] = position[index, :]                    
                if k==1 and j==1:
                    co1+=1
                    indbr1[co1]=index
                if k==2 and j==1:
                    co2+=1
                    indbr2[co2]=index 

index_backbone_and_branch_BCP=index
#anionposition = anionposition[anionposition[:, 0] != 0]

#generate the info of homopolymer
for i in range(1,N_homo+1):
    chptmp = chp[i]
    for j in range(1,L_homo+1):
        index = index + 1
        indexbbcount+=1
        atomid[index] = index
        chainid[index] = i+N_polymer
        charge[index] = 0
        typeid[index] = bcpbackboneAty
        position[index, :] = chptmp[j, :]
        imagenumber[index, :] = [0, 0, 0]
        indbb[indexbbcount]=index
index_homo_backbone=index

for i in range(1,N_homo+1):
    chptmp = chp[i]
    tmp = 0
    for k1 in range(1,int(L_homo / (Branchinterval_homo + 1))+1):
        for k in range(1,N_rep+1):
            for j in range(1,L_branch_homo+1):
                tmp = tmp + 1
                index = index + 1
                atomid[index] = index
                chainid[index] = i+N_polymer
                charge[index] = 0
                typeid[index] = bcpbranchty
                position[index, :] = chptmp[L_homo + tmp, :]
                imagenumber[index, :] = [0, 0, 0]
                if k==1 and j==1:
                    co1+=1
                    indbr1[co1]=index
                if k==2 and j==1:
                    co2+=1
                    indbr2[co2]=index 
index_homo_branch=index
    
N_cation = n_anion
##################################### build Li corresponding to the branched anions
for i in range(1,N_cation+1):
    index = index + 1
    atomid[index] = index
    chainid[index] = N_polymer + N_homo + i
    charge[index] = 1
    typeid[index] = cation_ty

for i in range(1,N_cation+1):
    [dx, dy, dz]=randomwalk(bondb)
    position[index - (i-1), :] = anionposition[i, :] + [dx, dy, dz] 
    imagenumber[index - (i-1), :] = [0, 0, 0]

###################################
# build freeions including cations and anions
freeions = int(freeions)
index_anion=index
if antype =='TFSI' or 'TF' or 'PEC':
    for i in range(1,freeions+1):
        index = index + 1
        atomid[index] = index
        chainid[index] = N_polymer + N_homo + N_cation + i    
        if antype =='TFSI':
            charge[index] = -0.13
            typeid[index] = tfsi_atom_a_type
        if antype =='TF':
            charge[index] = -0.13
            typeid[index] = tf_atom_a_type
        if antype =='PEC':
            charge[index] = -1
            typeid[index] = pec_atom_type
        
        position[index, :] = [np.random.rand() * L,np.random.rand() * L,np.random.rand() * L]
        imagenumber[index, :] = [0, 0, 0]   

if antype =='TFSI' or 'TF':
    for i in range(1,freeions+1):
        [dx,dy,dz]=randomwalk(bondc)
        index = index + 1
        atomid[index] = index
        chainid[index] = N_polymer + N_homo + N_cation + i
        if antype =='TFSI':
            charge[index] = -0.74
            typeid[index] = tfsi_atom_b_type
        if antype =='TF':
            charge[index] = -0.87
            typeid[index] = tf_atom_b_type

        position[index, :] = position[index-freeions,:]+ [dx, dy, dz] 
        imagenumber[index, :] = [0, 0, 0]

if antype =='TFSI':
    for i in range(1,freeions+1):
        [dx,dy,dz]=randomwalk(bondc)
        index = index + 1
        atomid[index] = index
        chainid[index] = N_polymer + N_homo + N_cation + i
        charge[index] = -0.13
        typeid[index] = tfsi_atom_a_type
        position[index, :] = position[index-freeions,:]+ [dx, dy, dz] 
        imagenumber[index, :] = [0, 0, 0]

for i in range(1,freeions+1):
    index = index + 1
    atomid[index] = index
    chainid[index] = N_polymer  + N_cation + N_homo + freeions + i  # # of polymer + # of Li corresponding to branched anion + # of homopolymer + # of free anions
    charge[index] = 1
    typeid[index] = cation_ty
    position[index, :] = position[index-freeions,:]+ [dx, dy, dz] 
    imagenumber[index, :] = [0, 0, 0]


####################################
#generate the bond parameters
bi = 0
est = (L_polymer - 1) *N_polymer +  N_rep  * L_branch_poly * int(LA / (Branchinterval_poly + 1))     * N_polymer \
     +(L_homo - 1)    *N_homo    +  N_rep  * L_branch_homo * int(L_homo / (Branchinterval_homo + 1)) * N_homo +  1\
     +freeions*2

A1 = np.zeros(est)
A2 = np.zeros(est)
B1 = np.zeros(est)
Btype = np.zeros(est)
check = np.zeros(est)
check2 = np.zeros(est)
#generate the bond info of backbone of BCP
for i in range(1,N_polymer+1):
    for j in range(1,1 + L_polymer - 1):
        bi = bi + 1
        if j == 1:
            A1[bi] = 1 + (i - 1) * L_polymer
            A2[bi] = A1[bi] + 1
        if j > 1:
            A1[bi] = A2[bi-1]
            A2[bi] = A1[bi] + 1
        check[bi] =     (position[int(A1[bi]), 0] - position[int(A2[bi]), 0])**2 + \
                        (position[int(A1[bi]), 1] - position[int(A2[bi]), 1])**2 + \
                        (position[int(A1[bi]), 2] - position[int(A2[bi]), 2])**2
        B1[bi] = bi
        if j > LA:
            Btype[bi] = bcpbackboneBbty
        else:
            Btype[bi] = bcpbackboneAbty

cp_polymer = bi

#generate the bond info of branch of BCP                    
for i in range(1,N_polymer+1):
    for j in range(1,int(LA / (Branchinterval_poly + 1))+1):
        Branch_position = 1 + (i - 1) * L_polymer + (j - 1) * (Branchinterval_poly + 1)
        for l in range(1,N_rep+1):
            for k in range(1,L_branch_poly + 1):
                bi = bi + 1
                if k == 1:
                    A1[bi] = Branch_position
                    if i == 1 and j == 1:
                        A2[bi] = Branch_position + indexb + (l - 1) * L_branch_poly
                    else:
                        A2[bi] = A2[bi-1] + 1
                    Btype[bi]=homo_eo_mmaty                    
                else:
                    A1[bi] = A2[bi-1]
                    A2[bi] = A1[bi] + 1
                    Btype[bi] = anionbty
                    if k==2:
                        Btype[bi]=abtyp
                    if k==3:
                        Btype[bi]=anionbty
                check[bi] = (position[int(A1[bi]), 0] - position[int(A2[bi]), 0])**2 + \
                            (position[int(A1[bi]), 1] - position[int(A2[bi]), 1])**2 + \
                            (position[int(A1[bi]), 2] - position[int(A2[bi]), 2])**2
                if  abs(check[bi] - bonda**2) > 0.1 and abs(check[bi] - bondb**2) > 0.1 and abs(check[bi] - bondc**2) > 0.1:
                    print('Warning: bond between %i and %i is wrong, length is: %.3f '% (int(A1[bi]),int(A2[bi]),check[bi]))

                #print('%.3f %.3f %.3f\n' % (int(A1[bi]), int(A2[bi]) ,check[bi]))
                B1[bi] = bi
                

cp_bcp_branch = bi

#generate the bond info of backbone of HP

for i in range(1,N_homo+1):
    for j in range(1,1 + L_homo - 1):
        bi = bi + 1
        if j == 1:
            A1[bi] = 1 + (i - 1) * L_homo  + index_backbone_and_branch_BCP
            A2[bi] = A1[bi] + 1 
        if j > 1:
            A1[bi] = A2[bi-1]
            A2[bi] = A1[bi] + 1
        check[bi] =     (position[int(A1[bi]), 0] - position[int(A2[bi]), 0])**2 + \
                        (position[int(A1[bi]), 1] - position[int(A2[bi]), 1])**2 + \
                        (position[int(A1[bi]), 2] - position[int(A2[bi]), 2])**2
        if  abs(check[bi] - bonda**2) > 0.1 and abs(check[bi] - bondb**2) > 0.1 and abs(check[bi] - bondc**2) > 0.1:
            print('Warning: bond between %i and %i is wrong, length is: %.3f '% (int(A1[bi]),int(A2[bi]),check[bi]))
        B1[bi] = bi
        Btype[bi] = bcpbackboneAbty   


#generate the bond info of branch of HP                    
for i in range(1,N_homo+1):
    for j in range(1,int(L_homo / (Branchinterval_homo + 1))+1):
        Branch_position = 1 + (i - 1) * L_homo + (j - 1) * (Branchinterval_homo + 1) + index_backbone_and_branch_BCP
        for l in range(1,N_rep+1):
            for k in range(1,L_branch_homo + 1):
                bi = bi + 1
                if k == 1:
                    A1[bi] = Branch_position
                    if i == 1 and j == 1:
                        A2[bi] = Branch_position + N_homo*L_homo + (l - 1) * L_branch_homo
                    else:
                        A2[bi] = A2[bi-1] + 1
                    Btype[bi]=homo_eo_mmaty                    
                else:
                    A1[bi] = A2[bi-1]
                    A2[bi] = A1[bi] + 1
                    Btype[bi] = homo_eo_bty
                check[bi] = (position[int(A1[bi]), 0] - position[int(A2[bi]), 0])**2 + \
                            (position[int(A1[bi]), 1] - position[int(A2[bi]), 1])**2 + \
                            (position[int(A1[bi]), 2] - position[int(A2[bi]), 2])**2
                if  abs(check[bi] - bonda**2) > 0.1 and abs(check[bi] - bondb**2) > 0.1 and abs(check[bi] - bondc**2) > 0.1:
                    print('Warning: bond between %i and %i is wrong, length is: %.3f '% (int(A1[bi]),int(A2[bi]),check[bi]))

                #print('%.3f %.3f %.3f\n' % (int(A1[bi]), int(A2[bi]) ,check[bi]))
                B1[bi] = bi
                 
#######################################################
if antype =='TFSI' or 'TF':
    for i in range(1,freeions+1):
        bi=bi+1
        A1[bi]=index_anion+i
        A2[bi]=index_anion+freeions+i
        check[bi] =     (position[int(A1[bi]), 0] - position[int(A2[bi]), 0])**2 + \
                        (position[int(A1[bi]), 1] - position[int(A2[bi]), 1])**2 + \
                        (position[int(A1[bi]), 2] - position[int(A2[bi]), 2])**2
        B1[bi]=bi
        Btype[bi]=anionbty
if antype =='TFSI':
    for i in range(1,freeions+1):
        bi=bi+1
        A1[bi]=index_anion+freeions+i
        A2[bi]=index_anion+freeions + freeions + i
        check[bi] =     (position[int(A1[bi]), 0] - position[int(A2[bi]), 0])**2 + \
                        (position[int(A1[bi]), 1] - position[int(A2[bi]), 1])**2 + \
                        (position[int(A1[bi]), 2] - position[int(A2[bi]), 2])**2
        B1[bi]=bi
        Btype[bi]=anionbty



######################################################
ai = 0
estangle = overall+2*(L_polymer-1)*N_polymer+2*(L_homo-1)*N_homo
An1 = np.zeros(estangle)
An2 = np.zeros(estangle)
An3 = np.zeros(estangle)
ai1 = np.zeros(estangle)
at = np.zeros(estangle)
#generate the angle info of branch of BCP  
for i in range(1,N_polymer+1):
    for j in range(1,1 + L_polymer - 2):
        ai = ai + 1
        if j == 1:
            An1[ai] = 1 + (i - 1) * L_polymer
            An2[ai] = An1[ai] + 1
            An3[ai] = An2[ai] + 1
        if j > 1:
            An1[ai] = An2[ai-1]
            An2[ai] = An1[ai] + 1
            An3[ai] = An2[ai] + 1
        at[ai] = bcpbackboneBaty
        ai1[ai] = ai
tmpa1=ai        
#generate the angle info of branch of BCP
for i in range(1,N_polymer+1):
    for j in range(1,int(LA / (Branchinterval_poly + 1))+1):
        Branch_position = 1 + (i - 1) * L_polymer + (j - 1) * (Branchinterval_poly + 1)
        for l in range(1,N_rep+1):
            for k in range(1,1+ L_branch_poly - 1):
                ai = ai + 1
                if k == 1:
                    An1[ai] = Branch_position
                    if i == 1 and j == 1:
                        An2[ai] = Branch_position + indexb + (l - 1) * L_branch_poly
                    else:
                        An2[ai] = An3[ai-1] + 1
                    An3[ai] = An2[ai] + 1
                    at[ai] = bcpbranchaty_singleion
                else:
                    An1[ai] = An2[ai-1]
                    An2[ai] = An1[ai] + 1
                    An3[ai] = An2[ai] + 1
                    at[ai] = bcpbranchaty_singleion
                ai1[ai] = ai
tmpa2=ai 
#generate the angle info of branch of HP 
for i in range(1,N_homo+1):
    for j in range(1,1 + L_homo - 2):
        ai = ai + 1
        if j == 1:
            An1[ai] = 1 + (i - 1) * L_homo + index_backbone_and_branch_BCP
            An2[ai] = An1[ai] + 1
            An3[ai] = An2[ai] + 1
        if j > 1:
            An1[ai] = An2[ai-1]
            An2[ai] = An1[ai] + 1
            An3[ai] = An2[ai] + 1
        at[ai] = bcpbackboneBaty
        ai1[ai] = ai
tmpa3=ai    
#generate the angle info of branch of HP
for i in range(1,N_homo+1):
    for j in range(1,int(L_homo/ (Branchinterval_homo + 1))+1):
        Branch_position = 1 + (i - 1) * L_homo + (j - 1) * (Branchinterval_homo + 1) + index_backbone_and_branch_BCP
        for l in range(1,N_rep+1):
            for k in range(1,1+ L_branch_homo - 1):
                ai = ai + 1
                if k == 1:
                    An1[ai] = Branch_position
                    if i == 1 and j == 1:
                        An2[ai] = Branch_position + L_homo*N_homo + (l - 1) * L_branch_homo
                    else:
                        An2[ai] = An3[ai-1] + 1
                    An3[ai] = An2[ai] + 1
                    at[ai] = bcpbranchaty
                else:
                    An1[ai] = An2[ai-1]
                    An2[ai] = An1[ai] + 1
                    An3[ai] = An2[ai] + 1
                    at[ai] = bcpbranchaty
                ai1[ai] = ai


cp_bcp_branch = ai
checkangle1=np.zeros(overall)
checkangle2=np.zeros(overall)
for i in range(1,1+ai):
    checkangle1[i] = (position[int(An1[i]), 0] - position[int(An2[i]), 0])**2 + \
                     (position[int(An1[i]), 1] - position[int(An2[i]), 1])**2 + \
                     (position[int(An1[i]), 2] - position[int(An2[i]), 2])**2
    checkangle2[i] = (position[int(An2[i]), 0] - position[int(An3[i]), 0])**2 + \
                     (position[int(An2[i]), 1] - position[int(An3[i]), 1])**2 + \
                     (position[int(An2[i]), 2] - position[int(An3[i]), 2])**2
    if  abs(checkangle1[i] - bonda**2) > 0.1 and abs(checkangle1[i] - bondb**2) > 0.1 and abs(checkangle1[i] - bondc**2) > 0.1:
        print('Warning: angle between %i and %i is wrong, length is: %.3f '% (int(An1[i]),int(An2[i]),checkangle1[i]))

#########################################################
if antype =='TFSI':
    for i in range(1,freeions+1):
        ai = ai + 1
        An1[ai] = index_anion + i
        An2[ai] = An1[ai] + freeions
        An3[ai] = An2[ai] + freeions
        at[ai] = bcpbranchaty
        ai1[ai] = ai

cp_bcp_branch = ai
checkangle1=np.zeros(overall)
checkangle2=np.zeros(overall)

##############################################generate the angle info of eo-mma-eo bond
if eommaeo:
    for i in range(1,N_polymer*L_polymer+N_homo*L_homo+1):
        ai = ai + 1
        An1[ai]=indbr1[i]
        An2[ai]=indbb[i]
        An3[ai]=indbr2[i]
        at[ai] = eommaeoaty
        ai1[ai] = ai


##############################################generate the angle info of mma-mma-eo bond
if mmammaeo:
    for k in range(1,N_rep+1):
        for i in range(1,N_polymer+1):
            for j in range(1,L_polymer):
                ai = ai + 1
                if j ==1:
                    An1[ai] = 1 + (i - 1) * L_polymer
                    An2[ai] = An1[ai] + 1
                if j > 1:
                    An1[ai] = An2[ai-1]
                    An2[ai] = An1[ai] + 1
                tmp = int(An2[ai])
                if k==1:
                    An3[ai] = indbr1[tmp]
                if k==2:
                    An3[ai] = indbr2[tmp]
                at[ai] = mmammaeoaty
                ai1[ai] = ai

    for k in range(1,N_rep+1):
        for i in range(1,N_homo+1):
            for j in range(1,L_homo):
                ai = ai + 1
                if j ==1:
                    An1[ai] = 1 + (i - 1) * L_homo + index_backbone_and_branch_BCP
                    An2[ai] = An1[ai] + 1
                if j > 1:
                    An1[ai] = An2[ai-1]
                    An2[ai] = An1[ai] + 1
                tmp = int(An2[ai]) - index_backbone_and_branch_BCP+N_polymer*L_polymer
                if k==1:
                    An3[ai] = indbr1[tmp]
                if k==2:
                    An3[ai] = indbr2[tmp]
                at[ai] = mmammaeoaty
                ai1[ai] = ai


for i in range(1,1+ai):
    checkangle1[i] = (position[int(An1[i]), 0] - position[int(An2[i]), 0])**2 + \
                     (position[int(An1[i]), 1] - position[int(An2[i]), 1])**2 + \
                     (position[int(An1[i]), 2] - position[int(An2[i]), 2])**2
    checkangle2[i] = (position[int(An2[i]), 0] - position[int(An3[i]), 0])**2 + \
                     (position[int(An2[i]), 1] - position[int(An3[i]), 1])**2 + \
                     (position[int(An2[i]), 2] - position[int(An3[i]), 2])**2
    if  abs(checkangle1[i] - bondb**2) > 0.1 and abs(checkangle1[i] - bondc**2) > 0.1:
        print('Warning: angle between %i and %i is wrong, length is: %.3f '% (int(An1[i]),int(An2[i]),checkangle1[i]))



with open('bulk.lammps'+str(N_polymer)+'_NP'+str(L_polymer)+'_BB', 'w') as file:
 
    file.write('\n')
    file.write('#btype: 1 ~ EO on POEM; 2 ~ PMMA bond type; 3 ~ anion bond type; 4 ~ EO-MMA bond type; 5 ~ TF-MMA bond type\n')


    file.write('#Number of N-:            %6d\n' %(n_anion))
    file.write('#Number of Li (branch):   %6d\n' %(N_cation))
    file.write('#Number of Li (free):     %6d\n' %(freeions))
    file.write('#EO/Li:                   %6d\n' %(EOLi))
    file.write('#real EO/Li:              %6.3f\n'%(realratio))
    file.write('#Number of PLiMTFSI:      %6d\n' %(N_polymer))
    file.write('#Chainlength of PLiMTFSI: %6d\n' %(L_polymer))
    file.write('#homopolymer length:      %6d\n' %(L_homo))
    file.write('#homopolymer number:      %6d\n' %(N_homo))
    file.write('#EO amount:               %6d\n' %(N_homo*EOperchain))

    file.write('\n')
    file.write('\n')
    file.write('{:>8}     atoms\n'.format(N_total))
    file.write('{:>8}     bonds\n'.format(bi))
    file.write('{:>8}     angles\n'.format(ai))
    file.write('{:>8}     dihedrals\n'.format(0))
    file.write('\n')

    file.write('{:>8}     atom types\n'.format(int(max(typeid))))
    file.write('{:>8}     bond types\n'.format(int(max(Btype))))
    file.write('{:>8}     angle types\n'.format(int(max(at))))
    file.write('{:>8}     dihedral types\n'.format(0))
    file.write('\n')

    file.write('{:>16.3f} {:>8.3f} xlo xhi\n'.format(0, float(L)))    
    file.write('{:>16.3f} {:>8.3f} ylo yhi\n'.format(0, float(L)))
    file.write('{:>16.3f} {:>8.3f} zlo zhi\n'.format(0, float(L)))
    file.write('\n')
    file.write('Atoms\n')
    file.write('\n')

    for i in range(1,overall):
        file.write('{:>8} {:>8} {:>8} {:>8.2f} {:>8.3f} {:>8.3f} {:>8.3f} {:>8d} {:>8d} {:>8d}\n'.format(int(atomid[i]),
                   int(chainid[i]), int(typeid[i]), float(charge[i]), position[i, 0], position[i, 1], position[i, 2], imagenumber[i, 0],
                   imagenumber[i, 1], imagenumber[i, 2]))

    file.write('\n')
    file.write('Bonds\n')
    file.write('\n')

    for i in range(1,bi+1):
        file.write('{:>8} {:>8} {:>8} {:>8}\n'.format(int(B1[i]), int(Btype[i]), int(A1[i]), int(A2[i])))

    file.write('\n')
    file.write('Angles\n')
    file.write('\n')

    for i in range(1,ai+1):
        file.write('{:>8} {:>8} {:>8} {:>8} {:>8}\n'.format(int(ai1[i]), int(at[i]), int(An1[i]), int(An2[i]), int(An3[i])))
    
        
    file.write('\n')
    file.write('Masses\n')
    file.write('\n')
    for i in range(1,int(max(typeid))+1):
        file.write('     %i  %.2f\n'% (i,float(mass[i])))
