<h1 align="left">
  <br>
  <img src="./img/hei-en.png" alt="HEI-Vs Logo" width="350">
  <br>
  HEI-Vs Engineering School - Industrial Automation Base
  <br>
</h1>

Cours AutB

Author: [Cédric Lenoir](mailto:cedric.lenoir@hevs.ch)
> Version 2026, V1.0 

# LAB 04 Programmation d'un bloc fonctionnel (FB) qui gère un actionneur

Dans ce travail, nous allons :
1. Programmer deux blocs fonctionnels (FB) basés sur le modèle **Execute** pour commander l'ouverture et la fermeture d'une pince.
2. Programmer un bloc fonctionnel (FB) basé sur le modèle **In Operation Base** pour connaitre l'état d'une pince.


<br>

<u>Pour rappel </u> : 

- Le modèle **"Execute"** fonctionne sur le principe d'un déclenchement ponctuel.
- Le modèle **"In Operation Base"** fonctionne de manière continue.

  
<br>



# Description de la pince
La pince que nous utiliserons dans le cadre de ce travail pratique est fabriquée par l'entreprise Schunk.
Elle est commandée en ouverture et en fermeture avec un électro-distributeur 5/2 monostable.

<u>Référence de la pince </u> : MPG + 40 (305521)

<figure>
    <img src="./img/pinceSchunk.png"
         alt="Lost image : MPG + 40.webp", width = 400>
    <figcaption>Schunk MPG + 40</figcaption>
</figure>



## Données techniques


|Propriété                         |Caractéristique|
|-----------------------------------|-------------|
|Course par doigt| 6 mm|
|Force à la fermeture| 135 N|
|Force à l'ouverture| 110 N|
|Température ambiant maxi.          | 90 °C|



<br>

# Description du capteur de la pince
La pince est dotée d'un capteur d'ouverture conçu par l'entreprise Schunk.
Ce capteur utilise la technologie à effet Hall pour mesurer avec précision l'ouverture de la pince.

<u>Référence du capteur </u> : MMS 22-IO link (0315830)


<figure>
    <img src="./img/capteurSchunk.png"
         alt="Lost image : MMS 22-IO-Link", width = 200>
    <figcaption>Schunk MMS 22-IO-Link</figcaption>
</figure>

<br>

Ce capteur repose sur l'interface de communication point à point **IO-Link** (conforme à la norme IEC 61131-9).

Contrairement aux systèmes traditionnels où la conversion analogique-numérique se fait au niveau de la carte d'entrées analogiques du PLC, celle-ci est effectuée directement à l'intérieur du capteur.


La seule donnée **synchrone** (Process data) transmise par ce capteur est la distance mesurée.

## Données techniques

|Propriété |Caractéristique|
|--------------|-------------|
|Description| Current process value|
|Data type| UIntegerT|
|Bit length| 16-bit|
|Bit offset| 0|
|Value range| 0 – 10000 (depending on the entered value for stroke per jaw)|
|Factor| 0.01|
|Offset| -|
|Unit| mm|



<br>

<u>Configuration du capteur utilisé au laboratoire d'automatisation </u> :

|Description | Valeur transmise par capteur|
|------------|-------------|
|Pince fermée | 1000|
|Pince ouverte | 0|






<br>


# Travail à réaliser

## Commande de la pince

<u>Objectif </u>:

Programmer deux blocs fonctionnels (FB) basés sur le modèle **Execute** qui commandent l'ouverture et la fermeture de la pince en respectant le diagramme d'état suivant : 



<figure>
    <img src="./img/ExecuteDoneBase.png"
         alt="Lost image :ExecuteDoneBase">
    <figcaption>Diagramme d'état "Execute"</figcaption>
</figure>


<br>

***Prérequis :***
Afin de désactiver la gestion de la pince qui est déjà implémentée dans le programme mis à disposition, supprimer ``PRG_DeviceManager`` sous Task Configuration --> MainTask (IEC-Tasks).

> Normally with last version, no more needed to delete ``PRG_DeviceManager``.

<br>

<figure>
    <img src="./img/PRG_DeviceManager.png"
         alt="Lost image :PRG_DeviceManager.png">
    <figcaption>Désactiver PRG_DeviceManager</figcaption>
</figure>


<br>

<u> Marche à suivre </u>:
1.  Programmer le DUT (Data User Type) ``E_ExecuteGripper`` de type ENUM (énumération) contenant les différents états possibles de la pince.

<br>

2.  Programmer les FB (Function Block) ``FB_OpenGripper`` et ``FB_CloseGripper`` en vous basant sur l'exemple de l'interface ci-dessous.
   
<br>
   
<u> Interface du FB ``FB_OpenGripper`` </u> :

``` iec 61131-3

FUNCTION_BLOCK FB_OpenGripper

VAR_INPUT
	Execute: BOOL;
	thOpenMax: WORD := 50; // Threshold to reach for gripper open
END_VAR

VAR_IN_OUT
	hwSensor: UA_Schunk_mms;
	hwEV: UA_Festo;
END_VAR

VAR_OUTPUT
	Done: BOOL;
	Active: BOOL;
	Error: BOOL;
END_VAR

VAR
	tnCheckDone: TON; // Time for opening before error
	rExecute: R_TRIG;
	eExecuteGripper: E_ExecuteGripper;
END_VAR

```

<br>



**Remarque :**
La pince s'ouvre ou se ferme grâce à un électro-distributeur **monostable**.
De ce fait, sa commande doit rester à ``TRUE`` (ou à ``FALSE``) après l'exécution du FB correspondant.


<u>Exemple pour le FB ``FB_OpenGripper`` </u>:

```iecst

IF eExecuteGripper = E_ExecuteGripper.InOp THEN
	hwEV.SetOut := TRUE;
END_IF

```


<br>


Ne pas oublier d'appeler le bloc fonctionnel ``FB_OpenGripper`` et ``FB_CloseGripper`` depuis le programme principal ``PRG_Student`` !


```iecst
fbOpenGripper(hwEV := GVL_Abox.uaAboxInterface.uaSchunkGripper,
	  hwSensor := GVL_Abox.uaAboxInterface.uaSchunk,
	       Done => stTestFbGripperHmi.executeOpenDone);
			  
fbCloseGripper(hwEV := GVL_Abox.uaAboxInterface.uaSchunkGripper,
	   hwSensor := GVL_Abox.uaAboxInterface.uaSchunk,
	        Done => stTestFbGripperHmi.executeCloseDone);
```


<br>

3. Tester le fonctionnement des blocs fonctionnel (FB)

La vérification du fonctionnement des FB pourra être réalisée soit avec l'outil "Watch" ou soit via Node-RED avec un programme mis à disposition.


<u>Information </u>:
Vous pouvez créer un DUT (Data User Type) de type STRUCT pour vous aider à écrire sur le paramètre d'entrée ``Execute`` des FB.

Déclaration de la structure :
```iecst
TYPE ST_TestFbGripperHmi :
STRUCT
	openGripper			: BOOL;
	closeGripper			: BOOL;
END_STRUCT
END_TYPE
```

<br>
Utilisation de la variable structurée dans le programme :

```iecst
// Request to open / close gripper
fbOpenGripper.Execute := stTestFbGripperHmi.openGripper;
fbCloseGripper.Execute := stTestFbGripperHmi.closeGripper;
```



<br>

<u> Marche à suivre pour utiliser l'application "Node-RED" mise à disposition </u> :


1. Remplacer le fichier ``flows.json`` qui se trouve dans le répertoire C:\Users\[ton_nom_utilisateur]\.node-red par le fichier ``flows.json`` qui se trouve sous : autb-lab-04_2025\PracticalWork_04_Student\

<br>

2. Démarrer Node-RED  
- Ouvrir l'invite de commande (--> cmd.exe)
- Entrer la commande : node-red

<br>

3. Accèder à l'interface utilisateur de Node-RED  
- Ouvrir le navigateur
- Entrer l'URL : http://localhost:1880

<br>

4) Installer les modules supplémentaires pour l'application (si nécessaire !)

- Cliquer sur les 3 traits horizontaux en haut à droite
<figure>
	<img src="./img/OuvrirPalette.png"
		 alt="Node-Red Icon" width="300"> 
</figure>

--> Gérer la palette  

--> Sélectionner l'onglet "Installer" 

--> Installer les modules suivants :  

	- @flowfuse/node-red-dashboard
	- node-red-contrib-ctrlx-automation

<br>

5) Ouvrir le Dashboard


<figure>
	<img src="./img/StartDashboard2.png"
		 alt="Ouvrir le Dashboard ">
	<figcaption>Ouvrir le Dashboard</figcaption>
</figure>


<br>

<u> Résultat </u> :


Dashboard :
<figure>
    <img src="./img/DashboardForGripper.png"
         alt="Lost image :DashboardForGripper">
    <figcaption>Dashboard de Node-RED</figcaption>
</figure>

<br>
Code :
<figure>
    <img src="./img/NodeRedCodeForGripper.png"
         alt="Lost image :ENodeRedCodeForGripper">
    <figcaption>Programme de test dans Node-RED</figcaption>
</figure>






<br>

6. Appuyer sur les boutons ``Open`` et ``Close`` sur le Dashboard "Node-RED" pour valider le fonctionnement correct des blocs fonctionnels  ``FB_OpenGripper`` et ``FB_CloseGripper``.


<br>







<br>

## Etats de la pince

<u>Objectif </u>:

Programmer un bloc fonctionnel (FB) basé sur le modèle **In Operation Base** qui permet de connaitre l'état de la pince en respectant le diagramme d'état suivant : 



<figure>
    <img src="./img/EnableInOpBaseSubState.png"
         alt="Lost image :EnableInOpBaseSubState">
    <figcaption>In Operation Base avec sous-états</figcaption>
</figure>


<br>

<u> Marche à suivre </u>:
1.  Programmer les DUT (Data User Type) ``E_InOperationBaseGripper`` et ``E_InOpGripper`` de type ENUM (énumération) contenant les différents états possibles de la pince.
<u>Remarque</u> : Ajouter le sous-état ``isIdle`` (non représenté sur la machine d'état ci-dessus) dans le DUT ``E_InOpGripper``.

<br>

3.  Programmer le FB (Function Block) ``FB_GripperState`` en vous basant sur l'interface ci-dessous.


``` iec61131-3

FUNCTION_BLOCK FB_GripperState

VAR_INPUT
	/// Default Input 
	Enable: BOOL;
	/// User Defined Inputs 
	thOpen: WORD := 50;     // Threshold gripper open
	thClose: WORD := 950;   // Threshold gripper closed
	thPartMin: WORD := 800; // Threshold minimum part present
	thPartMax: WORD := 860; // Threshold maximum part present
END_VAR

VAR_IN_OUT
	hw: UA_Schunk_mms;
END_VAR

VAR_OUTPUT
	/// Default Outputs 
	InOperation: BOOL;
	Error: BOOL;
	/// User Outputs 
	IsOpen: BOOL;      // Clipper opened
	IsClosed: BOOL;    // Clipper closed
	PartPresent: BOOL; // Part present
END_VAR

VAR
	eInOperationBaseGripper: E_InOperationBaseGripper := E_InOperationBaseGripper.Idle;
	eInOpGripper: E_InOpGripper := E_InOpGripper.IsIdle;
	tonIdleCondition: TON;
END_VAR
```



<br>

**Remarques :**

- L'état de la pince doit être déterminé dans l'état ``Init`` avant de passer dans l'état ``InOp`` de la machine interne.
- Afin d'éviter de générer une erreur lorsque la pince est en mouvement, la temporisation ``tonIdleCondition`` peut être utilisée pour laisser le temps à la pince d'atteindre sa position finale.
- Les sorties **IsOpen**, **IsClosed** et **PartPresent** sont activées uniquement lorsque la machine d'état principale est en ``InOp``.
- L'état ``IsIdle`` sera ajouté à la machine d'état interne, par contre, il ne sera pas activé car la machine d'état interne est initialisée dans ``Init`` avec les états soit  ``IsOpen``, ``IsClosed`` ou ``PartPresent``.
- L'état ``IsIdle`` dans InOp conduit à une **erreur**.
  
Il est possible que l'état de la pince soit indéterminé, par exemple si la pression d'air est manquante. Dans ce cas, le signal d'erreur doit être activé.

<br>

Ne pas oublier d'appeler le bloc fonctionnel ``FB_GripperState`` depuis le programme principal ``PRG_Student`` !

```iecst
fbGripperState(Enable := TRUE,
	           hw := GVL_Abox.uaAboxInterface.uaSchunk);
```



<br>

3. Tester le fonctionnement du bloc fonctionnel (FB)

La vérification du fonctionnement du FB pourra être réalisée via un "Dashboard" implémenté avec Node-RED.

<u>Information </u>:
Vous pouvez compléter le DUT (Data User Type) de type STRUCT créer dans l'exercice précédent pour vous aider à lire les paramètres de sortie du FB.

Déclaration de la structure :
```iecst
TYPE ST_TestFbGripperHmi :
STRUCT
	openGripper			: BOOL;
	closeGripper			: BOOL;
	gripperStateClosed		: BOOL;
	gripperStateOpen		: BOOL;
	gripperStatePartPresent	   	: BOOL;
	gripperStateError		: BOOL;
	gripperStateInOp		: BOOL;
END_STRUCT
END_TYPE
```

<br>

Utilisation de la variable structurée dans le programme :
```iecst
stTestFbGripperHmi.gripperStateClosed := ...
stTestFbGripperHmi.gripperStateOpen := ...
stTestFbGripperHmi.gripperStatePartPresent := ...
stTestFbGripperHmi.gripperStateError := ...
stTestFbGripperHmi.gripperStateInOp := ...
```





<br>

<u> Marche à suivre pour utiliser l'application "Node-RED" mise à disposition </u> :
1. Effectuer la marche à suivre de l'exercice précédent "Commande de la pince" (point 1 à 5)
2. Enclencher l'installation à l'aide du panneau opérateur "Siemens"
   
- Sélectionner le mode "Manual" à l'aide du bouton ``Manual``.
- Mettre l'installation dans l'état "Execute" à l'aide des boutons ``Reset`` et ``Start``.

<figure>
	<img src="./img/Packml.png"
		 alt="IHM sur Panel Operator Siemens", width = 800>
	<figcaption>IHM sur Panneau Opérateur "Siemens"</figcaption>
</figure>


<br>

3. Déplacer la pince à l'aide du Dashboard "Node-RED" et appuyer sur les boutons ``Open`` et ``Close`` pour valider le fonctionnement correct du bloc fonctionnel ``FB_GripperState``.





<figure>
    <img src="./img/PinceEprouvette.jpg"
         alt="Lost image :PinceEprouvette", width=800>
    <figcaption>Test capteur pince avec éprouvette</figcaption>
</figure>


# Alarmes
Toute machine devrait être livrée avec un système d'alarme.
Une fois les bases testées, vous pouvez ajouter deux alarmes pour les deux types de commande, Open et Close.

Si la fermeture ou l'ouverture de la pince ne se fait pas correctement, la machine doit transmettre une information et passer en état **Stopped**.

Vous ajouter les deux Function Block suivants dans votre programme et vous les paramétrez correctement.

```iecst
VAR
	fbSetAlarm_OpenGripper  : FB_HEVS_SetAlarm;
	fbSetAlarm_CloseGripper : FB_HEVS_SetAlarm;
END_VAR

```

#### E_EventCategory
Permet de sélectionner le type d'événement généré par l'alarme.

```iecst
{attribute 'qualified_only'}
TYPE E_EventCategory :
(
	Warning := 0,
	Complete:= 6,
	Abort   := 4,
	Stop    := 3,
	Hold    := 2,
	Suspend := 1
) DINT := Warning;
END_TYPE
```

-	**bAckAlarmTrig** n'est pas implémenté actuellement au niveau interface utilisateur.
-	**bSetAlarm** est la condition qui déclenche l'alarme.


### ID
**<span style="color:red;">ID doit être unique</span>**, car c'est cette valeur que le système utilise pour trier les alarmes. C'est aussi un principe, pour une machine, chaque alarme possède un numéro d'identification unique, même si on peut ajouter un paramètre **Value** et éventuellement modifier une partie du texte. Il serait ainsi envisageable d'avoir la même alarme pour close et open avec une **Value** et un **Message** différent. Mais à condition que **Category** ne change pas.

### Code
```iecst
fbSetAlarm_OpenGripper(bSetAlarm := ...,
					   bAckAlarmTrig := FALSE,
					   ID := 5,
					   Value := 1,
					   Message := 'Your Alarm Text',
					   Category := E_EventCategory.Stop,
					   // Reference to plc time from PackTag
					   plcDateTimePack	:= PackTag.Admin.PLCDateTime,
					   // Link to PackTag Admin
					   stAdminAlarm := PackTag.Admin.Alarm,
					   stAdminAlarmHistory := PackTag.Admin.AlarmHistory);	

fbSetAlarm_CloseGripper(bSetAlarm := ...,
					    bAckAlarmTrig := FALSE,
					    ID := 6,
					    Value := 1,
					    Message := 'Yout alarm Text',
					    Category := E_EventCategory.Stop,
					    // Reference to plc time from PackTag
					    plcDateTimePack	:= PackTag.Admin.PLCDateTime,
					    // Link to PackTag Admin
					    stAdminAlarm := PackTag.Admin.Alarm,
					    stAdminAlarmHistory := PackTag.Admin.AlarmHistory);	

```

# Conclusion
Les modèles de blocs fonctionnels "Execute" et "In Operation Base" jouent un rôle clé dans la programmation des PLC selon la norme IEC 61131-3.

En résumé, le modèle "Execute" se concentre sur l'exécution ponctuelle de tâches spécifiques, tandis que le modèle "In Operation Base" gère des tâches de manière continue.


