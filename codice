clearvars
clc
close all

test_case =4;
%test plan:
%- case 1: with PV,CHP,BOILER,HP,TES,ESS
%- case 2: with   ,CHP,BOILER,HP,TES,ESS
%- case 3: with PV,   ,      ,HP,TES,ESS
%- case 4: with PV,CHP,BOILER,  ,TES,ESS
%- case 5: with PV,CHP,BOILER,HP,TES,
%- case 6: with PV,CHP,      ,  ,TES,ESS
%- case 7: with PV,CHP,BOILER,HP,   ,
%- case 8: with PV,CHP,BOILER,HP,   ,ESS
%control horizon
H=24;
%prediction horizon
e=24;
%number of familit within the building
n=5;
%installed capacity PV
IC= 15;
% Objective
nC = H; %Number of Continuous Variables
nB = H; %Number of Binary Variables
beta = 1;% penalty parameter 
% Build xtype vector
xtype = [repmat('C',1,5*nC),repmat('B',1,nB),repmat('C',1,nC),repmat('B',1,nB),...
    repmat('C',1,nC),repmat('B',1,nB),repmat('C',1,nC),repmat('B',1,nB),repmat('C',1,nC),...
    repmat('B',1,nB),repmat('C',1,nC),repmat('B',1,nB),repmat('C',1,nC),repmat('B',1,nB),repmat('C',1,nC),...
    repmat('B',1,nB),repmat('C',1,nC),repmat('B',1,nB),repmat('C',1,2*nC)];
% Cost coefficients 
COM = 0.015;% operational and maintenance cost of CHP
kf = ones(H,1)*0.08; %fuel cost
kb2 = ones(H,1)*0.120;  %electricity cost off peak (with taxes and VAT included)
        kb2(7:19) = 0.145 ;% electricity cost peak
        kb1= (repmat(kb2',1,366));
ks2 = ones(H,1)*0.020;  %electricity cost off peak (without taxes and VAT included)
        ks2(7:19) = 0.045 ;% electricity cost peak
        ks1= (repmat(ks2',1,366));
% non-controllable loads consumption profile
%% Setup the Import Options
opts = spreadsheetImportOptions("NumVariables", 1);

% Specify sheet and range
opts.Sheet = "Foglio1";
opts.DataRange = "E2:E8785";

% Specify column names and types
opts.VariableNames = "householdconsumptionKWh";
opts.SelectedVariableNames = "householdconsumptionKWh";
opts.VariableTypes = "double";

% Import the data
heatingwaterspace2015 = readtable("C:\Users\Windows 10\Desktop\matlab dataset\heatingwater&space2015.xlsx",...
    opts, "UseExcel", false);
%% Setup the Import Options
opts = spreadsheetImportOptions("NumVariables", 1);

% Specify sheet and range
opts.Sheet = "Foglio1";
opts.DataRange = "A1:A8784";

% Specify column names and types
opts.VariableNames = "VarName1";
opts.SelectedVariableNames = "VarName1";
opts.VariableTypes = "double";

% Import the data
dataset2015solarres = readtable("C:\Users\Windows 10\Desktop\matlab dataset\dataset 2015 solar res.xlsx", opts, "UseExcel", false);

% controllable loads parameters
%washing machine
xminn1 = 0.1*ones(H,1); % minimum required energy per slot
xminn1(12:15)= 0.4;
xminnn1= (repmat(xminn1',1,366));
xmax1 = 0.6*ones(H,1);% maximum required energy per slot
Xtot = 7.5;
%oven
xmin2 = zeros(H,1);
xmax2 = 0.5*ones(H,1);

X1tot = 2.5;
%dishwasher
xminn3 = 0.1*ones(H,1);
xminn3(12:15)=0.2;
xminnn3= (repmat(xminn3',1,366));
xmax3 = 0.5*ones(H,1);
X2tot = 6;
%cooktop
xmin4 = 0.1*ones(H,1);
xmax4 = 0.5*ones(H,1);
X3tot = 4;
% electrical storage parameters
switch test_case
    case {5,7}
        sp_max = 0;
    otherwise
        sp_max = 5; % maximum charging energy per slot
end

sm_max = sp_max; % maximum discharging energy per slot
Smax = 20; % maximum state of charge
S0 = 0; % initial state of charge
etaps=0.95; % charging battery efficiency 
etams=0.95; % discharging battery efficiency
% thermal storage parameters
switch test_case
    case {7,8}
        spv_max = 0;
    otherwise
        spv_max = 5; % maximum charging thermal sotrage per slot
end

smv_max = spv_max; % maximum discharging thermal storage energy per slot
Smaxv = 20; % maximum state of charge of thermal storage
S0v = 0; % initial state of charge of thermal storage
etapv=0.95; % charging thermal storage efficiency 
etamv=0.95; % discharging thermal storage efficiency
% importing energy parameters
gp_max = 8; %maximum importing energy
gpsoft = 0.8*gp_max; % maximum soft importing energy
% exporting energy parameters
gm_max = 8; % maximum exporting energy
gmsoft = 0.8*gm_max; % maximum soft exporting energy
% CHP parameters
etachp_tot = 0.9; % total CHP efficiency 
etachp_el = 0.20*etachp_tot; % electrical CHP efficiency
etachp_th = 0.80*etachp_tot; % thermal CHP efficiency
switch test_case
    case{3}
        fchp_max=0;
        fchp_min=0;
    otherwise
        fchp_max = 25; % maximum fuel burned in CHP
        fchp_min = 1.25; % minimum fuel burned in CHP
end

pchp_max = fchp_max*etachp_el;% maximum electrical generation CHP
pchp_min = fchp_min*etachp_el; % minimum electrical generaation CHP
qchp_max = fchp_max*etachp_th; % maximum thermal generation CHP
qchp_min = fchp_min*etachp_th; % minimum thermal generation CHP
% boiler parameters
etaboil = 1; % boiler efficiency
switch test_case
    case{3,6}
        fboil_max=0;
        fboil_min=0;
    otherwise
        fboil_max = 10; % maximum fuel burned in boiler
        fboil_min = 0.5; % minimum fuel burned in boiler
end

qboil_max = fboil_max*etaboil; %maximum boiler thermal generation 
qboil_min = fboil_min*etaboil; %minimum boiler thermal generation
% heat pump paramters
COP = 4; %coefficient of performance of the heat pump
switch test_case
    case{4,6}
        php_max=0;
        php_min=0;
    otherwise
        php_max = 10; % maximum heat pump electrical input 
        php_min = 1.5; % minimum heat pump electrical input
end

qhp_max = COP*php_max; % maximum heat pump thermal generation
qhp_min = COP*php_min; % minimum heat pump thermal generation
%--------------------------------------------------------------------------
% inizialitation variables
x=zeros(H,1);
x1=zeros(H,1);
x2=zeros(H,1);
x3=zeros(H,1);
gp=zeros(H,1);
deltagp=zeros(H,1);
gm=zeros(H,1);
deltagm=zeros(H,1);
php=zeros(H,1);
fchp=zeros(H,1);
fboil=zeros(H,1);
zsp=zeros(H,1);
zsm=zeros(H,1);
zvp=zeros(H,1);
zvm=zeros(H,1);
alfa1=zeros(H,1);
alfa2=zeros(H,1);
% receding horizon control
% non controllable loads consumption profile
bload0 = n*[table2array(loads2015)]' ;    
% thermal loads consumption profile
qheat0 = n*[table2array(heatingwaterspace2015)]' ;  
% RES production profile (with 15 KW installed capacity)
switch test_case
    case{1,3,4,5}
        res0 = IC*[table2array(dataset2015solarres)]' ;
    otherwise
        res0= 0*[table2array(dataset2015solarres)]';
end
  
%box uncertainty
deltab0 = 1.4*bload0; %uncertainty set non controllable load
deltaq0 = 1.4*qheat0; %uncertainty set heating demand
deltar0 = 1.4*res0; %undertainty set pv production
for j=0:(e-1)
     kb = kb1(j+1:j+H);
     ks= ks1(j+1:j+H); 
     xmin1= (xminnn1(j+1:j+H))';
     xmin3=(xminnn3(j+1:j+H))';
    bload1=[bload0(j+1:j+H)]'
    res1 = [res0(j+1:j+H)]'
    qheat1= [qheat0(j+1:j+H)]'
    deltab =[deltab0(j+1:j+H)]'
    deltaq =[deltaq0(j+1:j+H)]'
    deltar =[deltar0(j+1:j+H)]'
%robustness factors
lbload=1;%robustness factor non controllable loads
lr=1;%robustess factor pv production
lq=1;%robustness factor heating demand



% resolution through "gurobi"
%--------------------------------------------------------------------------
% solve the quadratic programming problem:
%              min 0.5*x'*Q*x + f'*x    
%               x    
%              subject to:  A*x <= b
%                           Aeq*x = beq
%                           lb <= x <= ub
%
% Q and f
Kb = diag(kb);
Ks = diag(ks);
Q1 = [zeros(H*22,H*H)];
Q2 = [zeros(2*H,H*22)];
Q3 = [eye(H)*Kb*beta,zeros(H);zeros(H),(eye(H))*Ks*beta];
Q = [Q1;Q2,Q3];
f = [zeros(1,H*4),kb,zeros(1,H),-ks,zeros(1,H*3),(kf'+ones(1,H)*COM*etachp_tot),...
    zeros(1,H),kf',zeros(1,H*11)]';
%equality constraints 
%washing machine controllable load
Aeq_x = [ ones(1,H) zeros(1,23*H) ];
beq_x = Xtot;
%oven  controllable load
Aeq_x1 = [ zeros(1,H) ones(1,H) zeros(1,22*H) ];
beq_x1 = X1tot;
%dishwasher controllable load
Aeq_x2 = [ zeros(1,H*2) ones(1,H) zeros(1,21*H) ];
beq_x2 = X2tot;
%cooktop controllable load
Aeq_x3 = [ zeros(1,H*3) ones(1,H) zeros(1,20*H) ];
beq_x3 = X3tot;
%equality electrical balance, H equality constraints
Aeq_el = [eye(H), eye(H), eye(H), eye(H), -eye(H), zeros(H), eye(H), zeros(H),...
    eye(H),zeros(H),-eye(H)*etachp_el, zeros(H,3*H),eye(H),zeros(H),-eye(H),zeros(H,7*H)];
beq_el = [(res1-0.5*lr*deltar)-(bload1+0.5*lbload*deltab)];
%equality Thermal energy balance, H equality constraints
Aeq_th = [zeros(H,8*H),COP*eye(H),zeros(H), etachp_th*eye(H),zeros(H),...
    etaboil*eye(H),zeros(H,5*H),-eye(H),zeros(H),eye(H), zeros(H,3*H)];
beq_th = [qheat1+0.5*lq*deltaq];
%
Aeq = [ Aeq_x ; Aeq_x1;Aeq_x2; Aeq_x3; Aeq_el; Aeq_th ];
beq = [ beq_x ; beq_x1; beq_x2; beq_x3; beq_el; beq_th ];
%inequality constraints
%Maximum and minimum energy exchanged with utility grid
% inequality constraints g_p-delta_gp*gp_max <= 0 
A_gp1 = [zeros(H,4*H), eye(H), -gp_max*eye(H), zeros(H,18*H)];
b_gp1 = zeros(H,1);

% inequality constraints g_m-delta_gm*gm_max <= 0 
A_gm1 = [zeros(H,6*H), eye(H), -gm_max*eye(H), zeros(H,16*H)];
b_gm1 = zeros(H,1);

% inequality constraint delta_gp+delta_gm <= 1
A_gdelta = [zeros(H,5*H), eye(H),zeros(H),eye(H),zeros(H,16*H)];
b_gdelta = ones(H,1);
% Energy storage system modeling
% inequality constraints z_ps-delta_zp*sp_max <= 0 
A_zps1 = [zeros(H,14*H), eye(H), -sp_max*eye(H), zeros(H,8*H)];
b_zps1 = zeros(H,1);

% inequality constraints z_ms-delta_zm*sm_max <= 0 
A_zms1 = [zeros(H,16*H), eye(H), -sm_max*eye(H), zeros(H,6*H)];
b_zms1 = zeros(H,1);

% inequality constraint delta_zp+delta_zm <= 1
A_zsdelta = [zeros(H,15*H), eye(H),zeros(H),eye(H),zeros(H,6*H)];
b_zsdelta = ones(H,1);
% maximum battery storage capacity
% 2*H inequality constraints -> battery storage
A_1sp=[];  
for h=1:H
    a1sp = [ etaps*ones(1,h), zeros(1,H-h),zeros(1,H), -(1/etams)*ones(1,h), zeros(1,H-h) ];
    A_1sp=[A_1sp;a1sp];
end
A_2sp=[];
for h=1:H
    a2sp = [-etaps*ones(1,h), zeros(1,H-h),zeros(1,H), +(1/etams)*ones(1,h), zeros(1,H-h) ];
    A_2sp=[A_2sp;a2sp];
end
A_sp =[zeros(H,14*H),A_1sp,zeros(H,7*H); zeros(H,14*H), A_2sp,zeros(H,7*H)];
b_sp =[(Smax-(S0+sum(zsp)-sum(zsm)))*ones(H,1); (S0+sum(zsp)-sum(zsm))*ones(H,1) ];
%Micro CHP modeling

%inequality constraints fchp-delta_fchp*fchp_max <= 0 & -fchp+delta_fchp*fchp_min <= 0
A_fchp1 = [zeros(H,10*H), eye(H), -fchp_max*eye(H), zeros(H,12*H)];
b_fchp1 = zeros(H,1);
A_fchp2 = [zeros(H,10*H), -eye(H), fchp_min*eye(H) zeros(H,12*H)];
b_fchp2 = zeros(H,1);
%Heat pump modeling

%inequality constraints php-delta_php*php_max <= 0 & -php+delta_php*php_min <= 0
A_php1 = [zeros(H,8*H), eye(H), -php_max*eye(H), zeros(H,14*H)];
b_php1 = zeros(H,1);
A_php2 = [zeros(H,8*H), -eye(H), php_min*eye(H) zeros(H,14*H)];
b_php2 = zeros(H,1);
%Boiler

%inequality constraints fboil-delta_boil*fboil_max <= 0 & -fboil+delta_boil*fboil_min <= 0
A_fboil1 = [zeros(H,12*H), eye(H), -fboil_max*eye(H), zeros(H,10*H)];
b_fboil1 = zeros(H,1);
A_fboil2 = [zeros(H,12*H), -eye(H), fboil_min*eye(H) zeros(H,10*H)];
b_fboil2 = zeros(H,1);
%Thermal energy storage modeling
% inequality constraints z_pv-delta_pv*spv_max <= 0 
A_zpv1 = [zeros(H,18*H), eye(H), -spv_max*eye(H), zeros(H,4*H)];
b_zpv1 = zeros(H,1);

% inequality constraints z_mv-delta_mv*svm_max <= 0 
A_zmv1 = [zeros(H,20*H), eye(H), -smv_max*eye(H), zeros(H,2*H)];
b_zmv1 = zeros(H,1);

% inequality constraint delta_pv+delta_mv <= 1
A_zvdelta = [zeros(H,19*H), eye(H),zeros(H),eye(H),zeros(H,2*H)];
b_zvdelta = ones(H,1);
% maximum thermal energy storage capacity
% 2*H inequality constraints -> thermal storage
A_1vp=[];  
for h=1:H
    a1vp = [ etapv*ones(1,h), zeros(1,H-h),zeros(1,H), -(1/etamv)*ones(1,h), zeros(1,H-h) ];
    A_1vp=[A_1vp;a1vp];
end
A_2vp=[];
for h=1:H
    a2vp = [-etapv*ones(1,h), zeros(1,H-h),zeros(1,H), +(1/etamv)*ones(1,h), zeros(1,H-h) ];
    A_2vp=[A_2vp;a2vp];
end
A_vp =[zeros(H,18*H),A_1vp,zeros(H,3*H); zeros(H,18*H), A_2vp,zeros(H,3*H)];
b_vp =[(Smaxv-(S0v+sum(zvp)-sum(zvm)))*ones(H,1); (S0v+sum(zvp)-sum(zvm))*ones(H,1) ];
%inequality soft constraints: -alfa1+gp <= gp_soft
A_gpsoft = [zeros(H,4*H), eye(H), zeros(H,17*H), -eye(H), zeros(H)];
b_gpsoft = [ones(H,1)*gpsoft];
%inequality soft constraints: -alfa2+gm <= gm_soft
A_gmsoft = [zeros(H,6*H), eye(H), zeros(H,16*H), -eye(H)];
b_gmsoft = [ones(H,1)*gmsoft];
% A and b
A = [A_gp1;A_gm1; A_gdelta;A_zps1;A_zms1;...
    A_zsdelta;A_sp;A_fchp1;A_fchp2;...
    A_php1;A_php2; A_fboil1;A_fboil2;...
    A_zpv1; A_zmv1; A_zvdelta;A_vp;A_gpsoft;A_gmsoft];

b = [b_gp1;b_gm1; b_gdelta;b_zps1;b_zms1;...
    b_zsdelta;b_sp;b_fchp1;b_fchp2;...
    b_php1;b_php2; b_fboil1;b_fboil2;...
    b_zpv1; b_zmv1; b_zvdelta;b_vp;b_gpsoft;b_gmsoft];
%bounding constraints
% H bounding for each controllable load -> controllable loads
lb_x = xmin1;
ub_x = xmax1;
lb_x1 = xmin2;
ub_x1 = xmax2;
lb_x2 = xmin3;
ub_x2 = xmax3;
lb_x3 = xmin4;
ub_x3 = xmax4;
%bounding 0<= gp <= gp_max
lb_gp = zeros(H,1);
ub_gp = ones(H,1)*gp_max;
%bounding 0<= deltagp <= 1
lb_deltagp = zeros(H,1);
ub_deltagp = ones(H,1);
%bounding 0<= gm <= gm_max
lb_gm = zeros(H,1);
ub_gm = ones(H,1)*gm_max;
%BOUNDING 0<= deltagm <= 1
lb_deltagm= zeros(H,1);
ub_deltagm= ones(H,1);
%bounding php_min<= php <= php_max
lb_php = -inf(H,1);
ub_php = php_max*ones(H,1);
%bounding 0<= deltaphp <= 1
lb_deltaphp = zeros(H,1);
ub_deltaphp = ones(H,1);
%bounding fchp_min<= fchp <= fchp_max
lb_fchp = -inf(H,1);
ub_fchp = fchp_max*ones(H,1);
%bounding 0<= deltafchp <= 1
lb_deltafchp = zeros(H,1);
ub_deltafchp = ones(H,1);
%bounding fboil_min<= fboil <= fboil_max
lb_fboil = -inf(H,1);
ub_fboil = fboil_max*ones(H,1);
%bounding 0<= deltafboil <= 1
lb_deltafboil = zeros(H,1);
ub_deltafboil = ones(H,1);
%bounding zsp_min<= zsp <= zsp_max
lb_zsp = zeros(H,1);
ub_zsp = sp_max*ones(H,1);
%bounding 0<= deltazsp <= 1
lb_deltazsp = zeros(H,1);
ub_deltazsp = ones(H,1);
%bounding zsm_min<= zsm <= zsm_max
lb_zsm = zeros(H,1);
ub_zsm = sm_max*ones(H,1);
%bounding 0<= deltazsp <= 1
lb_deltazsm = zeros(H,1);
ub_deltazsm = ones(H,1);
%bounding zvp_min<= zvp <= zvp_max
lb_zvp = zeros(H,1);
ub_zvp = spv_max*ones(H,1);
%bounding 0<= deltazvp <= 1
lb_deltazvp = zeros(H,1);
ub_deltazvp = ones(H,1);
%bounding zvm_min<= zvm <= zvm_max
lb_zvm = zeros(H,1);
ub_zvm = smv_max*ones(H,1);
%bounding 0<= deltazvp <= 1
lb_deltazvm = zeros(H,1);
ub_deltazvm = ones(H,1);
%bounding  soft constraints 0<= alfa1 <= +inf
lb_alfa1 = zeros(H,1);
ub_alfa1 = inf(H,1);
%bounding  soft constraints 0<= alfa2 <= +inf
lb_alfa2 = zeros(H,1);
ub_alfa2 = inf(H,1);
% lb & ub
lb = [lb_x;lb_x1;lb_x2;lb_x3;lb_gp;lb_deltagp;lb_gm;lb_deltagm;...
    lb_php;lb_deltaphp;lb_fchp;lb_deltafchp;lb_fboil;lb_deltafboil;...
    lb_zsp;lb_deltazsp;lb_zsm;lb_deltazsm;lb_zvp;lb_deltazvp;lb_zvm;...
    lb_deltazvm;lb_alfa1;lb_alfa2 ];
ub = [ub_x;ub_x1;ub_x2;ub_x3;ub_gp;ub_deltagp;ub_gm;ub_deltagm;...
    ub_php;ub_deltaphp;ub_fchp;ub_deltafchp;ub_fboil;ub_deltafboil;...
    ub_zsp;ub_deltazsp;ub_zsm;ub_deltazsm;ub_zvp;ub_deltazvp;ub_zvm;...
    ub_deltazvm;ub_alfa1;ub_alfa2 ];

%Gurobi code
model.A = sparse([A; Aeq]);
model.rhs = [b; beq];
model.sense = [repmat('<',1,size(A,1)) repmat('=',1,size(Aeq,1))];
model.vtype = xtype;
model.obj = f;
model.lb = lb;
model.ub = ub;
model.Q = sparse(Q);

%gurobi optimization
result = gurobi(model)
xval = result.x;



%--------------------------------------------------------------------------
% results analysis
%--------------------------------------------------------------------------
%input data
bload(j+1,:)=bload1(1)+0.5*lbload*deltab(1);
res(j+1,:)=res1(1)-0.5*lr*deltar(1);
qheat(j+1,:)=qheat1(1)+0.5*lq*deltaq(1);

%extract decision variables
% extract x
x(j+1,:) = xval(1);
% extract x1
x1(j+1,:) = xval(H+1);
% extract x2
x2(j+1,:) = xval(2*H+1);
% extract x3
x3(j+1,:) = xval(3*H+1);
% extract gp
gp(j+1,:) = xval(4*H+1);
% extract deltagp
deltagp = xval(5*H+1);
%extract gm
gm(j+1,:) = xval(6*H+1);
%extract deltagm
deltagm = xval(7*H+1);
%extract php
php(j+1,:) = xval(8*H+1);
%extract fchp
fchp(j+1,:) = xval(10*H+1);
%extract fboil
fboil(j+1,:) = xval(12*H+1);
%extract zsp
zsp(j+1,:) = xval(14*H+1);
%extract zsm
zsm(j+1,:) = xval(16*H+1);
%extract zvp
zvp(j+1,:) = xval(18*H+1);
%extract zvm
zvm(j+1,:) = xval(20*H+1);
%extract alfa1
alfa1(j+1,:) = xval(22*H+1);
alfa2(j+1,:) = xval(23*H+1);
end

xtot = x+x1+x2+x3;
%total cost
Ctot = kb1(1,1:e)*gp-ks1(1,1:e)*gm+((0.08+COM*etachp_tot)*ones(1,e))*fchp+...
    (0.08*ones(1,e))*fboil+beta*(((kb1(1,1:e)*(alfa1.^2))+(ks1(1,1:e)*(alfa2.^2))));
%MonteCarlo simulation (MC iterations)
%adding a normally distributed random sequence with zero mean and standard
%deviation equal to 0.2[KWh] to the nominal predicted value
%Truncated Gaussian with interval [0 inf], because isn't possible to have
%neagtive values
MC=1000;
for j=1:e
        b=bload1(j)+TruncatedGaussian(0.2,[-inf inf]-bload1(j),[MC 1])
        if res(j)>0
        r=res1(j)+TruncatedGaussian(0.2,[0 inf]-res1(j),[MC 1])
        else 
            r=zeros(MC,1)
        end
        q=qheat1(j)+TruncatedGaussian(0.2,[-inf inf]-qheat1(j),[MC 1])
        matb(j,:)=b
        matres(j,:)=r
        matq(j,:)=q
         
end
matb(matb<0) =0;
matq(matq<0) =0;
for i=1:MC
  mel=-(x+x1+x2+x3)-matb(:,i)+fchp*etachp_el-php+matres(:,i)-zsp+zsm+deltagp*gp_max-deltagm*gm_max;  
  matel(i,:)=mel
  mq= fboil*etaboil+fchp*etachp_th+php*COP-zvp+zvm-matq(:,i)
  matth(i,:)=mq
end
%extract violation electrical balance
cv= 0.20; % violation cost (EUR per KWh)
matel(matel >0) =0;
Cv = matel*(cv*ones(e,1));
cvaverage= -(sum(Cv))/MC;
% average violation
matel(matel<0) =1; %quante volte violo il vincolo mediamente all'interno del prediction horizon
VIOL= matel*ones(e,1);
violmedia= ((sum(VIOL))/(e*MC))*100;%valore in percentuale
%violation thermal balance
matth(matth >0) =0;
Cvth = matth*(cv*ones(e,1));
cvaverageth= -(sum(Cvth))/MC;
% average violation
matth(matth<0) =1; %quante volte violo il vincolo mediamente in 24 ore
VIOLth= matth*ones(e,1);
violmediath= ((sum(VIOLth))/(e*MC))*100; %valore in percentuale
%total cost considering average violation cost of electrical and thermal
%balance
CTOT= Ctot+cvaverage+cvaverageth;

% time slots
t = 1:e;
% figure plot
bar_width = 1;
h1 = figure(1);
hold on;
hpchp = bar( t, gp+zsm+res+fchp*etachp_el,  bar_width, 'FaceColor','k','EdgeColor','k' ) ;
hres = bar( t, gp+zsm+res,  bar_width, 'FaceColor','r','EdgeColor','k' ) ;
hzsm  = bar( t, gp+zsm, bar_width, 'FaceColor','c','EdgeColor','k' ) ;
hgp  = bar( t, gp, bar_width, 'FaceColor','b','EdgeColor','k' ) ;
hgm  = bar( t, -gm-xtot-bload-php-zsp, bar_width, 'FaceColor','b','EdgeColor','k' ) ;
hxtot  = bar( t,-xtot-bload-php-zsp, bar_width, 'FaceColor','y','EdgeColor','k' ) ;
hbload  = bar( t,-bload-php-zsp, bar_width, 'FaceColor','g','EdgeColor','k' ) ;
hphp = bar( t,-php-zsp, bar_width, 'FaceColor','m','EdgeColor','k' ) ;
hzsp = bar( t,-zsp, bar_width, 'FaceColor','c','EdgeColor','k' ) ;
xlabel( 'time slot', 'FontSize',12.5 ) ;
ylabel( {'energy profile of consumption / production /'; ...
    'storage activities in the smart home [KWh]'}, 'FontSize',12.5 ) ;
legend( [ hpchp hres hzsm hgp hgm hxtot hbload hphp hzsp], 'electrical generation chp',...
    'pv generation','battery discharging',...
    'importing energy','exporting energy',...
    'controllable loads','non controllable loads','electrical generation heat pump',...
    'battery charging','Location','SouthEast' ) ;
V = axis;
xmin = 0.5;  xmax = H+0.5;  
axis( [ xmin xmax V(3) V(4) ] ) ;

h2 = figure(2);
hold on;
hstop = bar( t, xtot, bar_width, 'FaceColor','y','EdgeColor','k' ) ;
xlabel( 'time slot', 'FontSize',12.5 ) ;
ylabel( {'controllable load profile of consumption [KWh]'}, 'FontSize',12.5 ) ;
legend( [  hstop ], 'controllable load','Location','SouthEast' ) ;
V = axis;
xmin = 0.5;  xmax = H+0.5;  
axis( [ xmin xmax V(3) V(4) ] ) ;

h3 = figure(3);
hold on;
hstop = bar( t, zsp, bar_width, 'FaceColor','c','EdgeColor','k' ) ;
hstom = bar( t, -zsm, bar_width, 'FaceColor','c','EdgeColor','k' ) ;
xlabel( 'time slot', 'FontSize',12.5 ) ;
ylabel( {'energy profile of'; 'electrical storage activities in the smart home [KWh]'}, 'FontSize',12.5 ) ;
legend( [  hstop ], 'electrical storage','Location','SouthEast' ) ;
V = axis;
xmin = 0.5;  xmax = H+0.5;  
axis( [ xmin xmax V(3) V(4) ] ) ;

h4 = figure(4);
hold on;
hstop = bar( t, zvp, bar_width, 'FaceColor','c','EdgeColor','k' ) ;
hstom = bar( t, -zvm, bar_width, 'FaceColor','c','EdgeColor','k' ) ;
xlabel( 'time slot', 'FontSize',12.5 ) ;
ylabel( {'energy profile of'; 'thermal storage activities in the smart home [KWh]'}, 'FontSize',12.5 ) ;
legend( [  hstop ], 'thermal storage','Location','SouthEast' ) ;
V = axis;
xmin = 0.5;  xmax = H+0.5;  
axis( [ xmin xmax V(3) V(4) ] ) ;

h5 = figure(5);
hold on;
hqhp = bar( t, php*COP+fchp*etachp_th+fboil*etaboil+zvm, bar_width, 'FaceColor','y','EdgeColor','k' ) ;
hqchp = bar( t,fchp*etachp_th+fboil*etaboil+zvm , bar_width, 'FaceColor','r','EdgeColor','k' ) ;
hqboil = bar( t,fboil*etaboil+zvm, bar_width, 'FaceColor','g','EdgeColor','k' ) ;
hzvm = bar( t,zvm, bar_width, 'FaceColor','c','EdgeColor','k' ) ;
hqheat = bar( t, -qheat-zvp, bar_width, 'FaceColor','m','EdgeColor','k' ) ;
hzvp = bar( t,-zvp, bar_width, 'FaceColor','c','EdgeColor','k' ) ;
xlabel( 'time slot', 'FontSize',12.5 ) ;
ylabel( {'thermal energy profile of consumption / production /'; ...
    'storage activities in the smart home [KWh]'}, 'FontSize',12.5 ) ;
legend( [  hqhp hqchp hqboil hzvm hqheat hzvp ], 'thermal energy generation heat pump',...
    'thermal energy generation chp','thermal energy generation boiler',...
    'thermal storage discharge','heat demand','thermal storage charge','Location','SouthEast' ) ;
V = axis;
xmin = 0.5;  xmax = H+0.5;  
axis( [ xmin xmax V(3) V(4) ] ) ;

h6 = figure(6);
hold on;
hgp = bar( t, gp, bar_width, 'FaceColor','b','EdgeColor','k' ) ;
hgm = bar( t, -gm, bar_width, 'FaceColor','c','EdgeColor','k' ) ;
xlabel( 'time slot', 'FontSize',12.5 ) ;
ylabel( {'energy profile of'; 'importing/exporting energy [KWh]'}, 'FontSize',12.5 ) ;
legend( [  hgp hgm ], 'importing energy','exporting energy','Location','SouthEast' ) ;
V = axis;
xmin = 0.5;  xmax = H+0.5;  
axis( [ xmin xmax V(3) V(4) ] ) ;
