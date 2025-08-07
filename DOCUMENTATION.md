# Документация проекта **Remplanner3D**

## 1. Общие сведения

- Все размеры задаются в **сантиметрах**.
- **Modules** создаются напрямую в приложении.
- **Systems** создаются динамическим импортом из конфигурационных файлов в зависимости от режима (`dev` / `prod`) — это помогает не раздувать итоговый бандл.


## 2. Примеры сценариев поведения

**2.1 Сдвиг текстуры пользователем**

1. [systems/**UserInterface**](https://gitlab.com/remplanner/visual/-/blob/master/js/3d/src/systems/UserInterface/UserInterface.js?ref_type=heads) вешает событие на инпут и при изменении пользователем свойства дергает метод в **PanelMaterials**
1. [systems/UserInterface/sections/**PanelMaterials**](https://gitlab.com/remplanner/visual/-/blob/master/js/3d/src/systems/UserInterface/sections/PanelMaterials.js?ref_type=heads) запрашивает у [systems/**Selector**](https://gitlab.com/remplanner/visual/-/blob/master/js/3d/src/systems/Selector/Selector.js?ref_type=heads) текущий фокус и отправляет событие *skinChangeTexture* медиатору.
1. [modules/**Skin**](https://gitlab.com/remplanner/visual/-/blob/master/js/3d/src/modules/Skin/Skin.js?ref_type=heads) перехватывает событие и передаёт его в **Entities**.
1. [system/Graphics/**Entities**](https://gitlab.com/remplanner/visual/-/blob/master/js/3d/src/Engine/systems/Graphics/Entities.js?ref_type=heads) меняет материал с помощью [system/Graphics/**Materials**](https://gitlab.com/remplanner/visual/-/blob/master/js/3d/src/Engine/systems/Graphics/Materials.js?ref_type=heads).
1. [modules/**Skin**](https://gitlab.com/remplanner/visual/-/blob/master/js/3d/src/modules/Skin/Skin.js?ref_type=heads) посылает событие *prjChange* в **Project**.
1. [modules/**Project**](https://gitlab.com/remplanner/visual/-/blob/master/js/3d/src/modules/Project/Project.js?ref_type=heads) фиксирует, что проект изменён.


**2.2. Добавление декора**

1. В [systems/**UserInterface**](https://gitlab.com/remplanner/visual/-/blob/master/js/3d/src/systems/UserInterface/UserInterface.js?ref_type=heads) на кнопку **Добавить декор** вешается обработчик, открывающий [systems/UserInterface/sections/**PanelDecor**](https://gitlab.com/remplanner/visual/-/blob/master/js/3d/src/systems/UserInterface/sections/PanelDecor.js?ref_type=heads).
1. На HTML‑элемент картинки декора через [systems/UserInterface/InterfaceActions](https://gitlab.com/remplanner/visual/-/blob/master/js/3d/src/systems/UserInterface/InterfaceActions.js?ref_type=heads) вешается событие *decorCreate*.
1. [modules/**Decor**](https://gitlab.com/remplanner/visual/-/blob/master/js/3d/src/modules/Decor/Decor.js?ref_type=heads) перехватывает *decorCreate*, подгружает модель через [modules/**Models**](https://gitlab.com/remplanner/visual/-/blob/master/js/3d/src/modules/Model/Model.js?ref_type=heads).
1. [modules/**Models**](https://gitlab.com/remplanner/visual/-/blob/master/js/3d/src/modules/Model/Model.js?ref_type=heads) бросает событие *tmpLoadGltf* в [Engine/systems/Graphics](https://gitlab.com/remplanner/visual/-/blob/master/js/3d/src/Engine/systems/Graphics/Graphics.js?ref_type=heads).
1. [modules/Decor/**Placement**](https://gitlab.com/remplanner/visual/-/blob/master/js/3d/src/modules/Decor/Placement/Placement.js?ref_type=heads) выравнивает модель с помощью [modules/Decor/**Lair**](https://gitlab.com/remplanner/visual/-/blob/master/js/3d/src/modules/Decor/Lair.js?ref_type=heads) и инициирует событие *plcDecorAdd*.
1. Далее [modules/Decor/**Placement**](https://gitlab.com/remplanner/visual/-/blob/master/js/3d/src/modules/Decor/Placement/Placement.js?ref_type=heads):
   - ждёт от [systems/**Selector**](https://gitlab.com/remplanner/visual/-/blob/master/js/3d/src/systems/Selector/Selector.js?ref_type=heads) смены активного 3D‑меша;
   - отправляет событие *grRaySetByPlane*;
   - переходит в режим *PLACE*.
1. [Engine/systems/Graphics/Raycast](https://gitlab.com/remplanner/visual/-/blob/master/js/3d/src/Engine/systems/Graphics/Raycast.js?ref_type=heads) задаёт поверхность для размещения.
1. [modules/Decor/**Placement**](https://gitlab.com/remplanner/visual/-/blob/master/js/3d/src/modules/Decor/Placement/Placement.js?ref_type=heads) в режиме **PLACE**  (подписка на `mouseMove`) двигает меш по квартире.
   - На `mouseClick` — если режим **PLACE**, перемещение завершается.
   - На `mouseDown` — если режим **NONE**, добавляется круг поворота.
   - В режиме **ROTATE** `mouseMove` вращает меш.
   - На `mouseUp` генерируется событие **`prjChange`** (сохранить проект).
1. [modules/**Project**](https://gitlab.com/remplanner/visual/-/blob/master/js/3d/src/modules/Project/Project.js?ref_type=heads) фиксирует изменение проекта.


### 3 Классы

**3.1 Основные файлы**

[**Remplanner3D**](https://gitlab.com/remplanner/visual/-/blob/master/js/3d/src/RPlanner3D.js?ref_type=heads)
- Точка точка входа в приложение
- Читает сонфиг [**RPlanner3D-cfg**](https://gitlab.com/remplanner/visual/-/blob/master/js/3d/src/RPlanner3D-cfg.js?ref_type=heads) приложения.
- Создаёт экземпляры **modules**:
   - [modules/**Application**](https://gitlab.com/remplanner/visual/-/blob/master/js/3d/src/modules/Application.js?ref_type=heads)
   - [modules/**Registry**](https://gitlab.com/remplanner/visual/-/blob/master/js/3d/src/modules/Registry.js?ref_type=heads)
   - [modules/**Project**](https://gitlab.com/remplanner/visual/-/blob/master/js/3d/src/modules/Project/Project.js?ref_type=heads)
   - [modules/**Mode**](https://gitlab.com/remplanner/visual/-/blob/master/js/3d/src/modules/Mode.js?ref_type=heads)
   - [modules/**Decor**](https://gitlab.com/remplanner/visual/-/blob/master/js/3d/src/modules/Decor/Decor.js?ref_type=heads)
   - [modules/**Skin**](https://gitlab.com/remplanner/visual/-/blob/master/js/3d/src/modules/Skin/Skin.js?ref_type=heads)
   - [modules/**Entourage**](https://gitlab.com/remplanner/visual/-/blob/master/js/3d/src/modules/Entourage/Entourage.js?ref_type=heads)
   - [modules/**Apartments**](https://gitlab.com/remplanner/visual/-/blob/master/js/3d/src/modules/Apartments.js?ref_type=heads)
   - [modules/**Spatium**](https://gitlab.com/remplanner/visual/-/blob/master/js/3d/src/modules/Spatium/Spatium.js?ref_type=heads)
   - [**Engine**](https://gitlab.com/remplanner/visual/-/blob/master/js/3d/src/Engine/Engine.js?ref_type=heads)


[**Engine**](https://gitlab.com/remplanner/visual/-/blob/master/js/3d/src/Engine/Engine.js?ref_type=heads) <a id="engine-anchor"></a>
- Создаётся точкой входа [Remplanner3D](https://gitlab.com/remplanner/visual/-/blob/master/js/3d/src/RPlanner3D.js?ref_type=heads).
- По конфигу [**RPlanner3D-cfg**](https://gitlab.com/remplanner/visual/-/blob/master/js/3d/src/RPlanner3D-cfg.js?ref_type=heads) создаёт все **systems** (например `systems/**UserInterface**`).
- Все `systems` изолированы и общаются через **медиатор**.


**3.2 Modules создаваемые в [Remplanner3D](https://gitlab.com/remplanner/visual/-/blob/master/js/3d/src/RPlanner3D.js?ref_type=heads)**

[modules/**Application**](https://gitlab.com/remplanner/visual/-/blob/master/js/3d/src/modules/Application.js?ref_type=heads)
- создается точкой входа Remplanner3D
- хранит права пользователя
- слушает ошибки приложения, отсылает на сервер репорты об ошибках
- создает эффекты постпроцессинга в зависимости от аппаратной поддержки
- `.isSandbox` = window['rplanner_sandbox'] флаг не слать алерты на реальный сервер


[modules/**Registry**](https://gitlab.com/remplanner/visual/-/blob/master/js/3d/src/modules/Registry.js?ref_type=heads)
- создается точкой входа Remplanner3D
- Реестр данных для моделей для добавления в квартиру пользователя. `**window.global_visual_data**` - серверные данные зашитые в страницу с бэка
- Отдает верстку для [systems/**UserInterface**](https://gitlab.com/remplanner/visual/-/blob/master/js/3d/src/systems/UserInterface/UserInterface.js?ref_type=heads) каталогов элементов


[modules/**Project**](https://gitlab.com/remplanner/visual/-/blob/master/js/3d/src/modules/Project/Project.js?ref_type=heads)
- создается точкой входа Remplanner3D
- создает класс [modules/Project/**ProjectSource**](https://gitlab.com/remplanner/visual/-/blob/master/js/3d/src/modules/Project/ProjectSource.js?ref_type=heads) в котором происходит загрузка и валидация проекта пользователя- хранит данные проекта квартиры пользователя


   [modules/Project/**ProjectSource**](https://gitlab.com/remplanner/visual/-/blob/master/js/3d/src/modules/Project/ProjectSource.js?ref_type=heads)
   - подгрузка и валидация проекта квартиры пользователя


   [modules/Project/**OldVersion**](https://gitlab.com/remplanner/visual/-/blob/master/js/3d/src/modules/Project/OldVersion.js?ref_type=heads)
   - конвертация серверных сохраненных данных в формат удобный приложению [подробнее](#oldversion-anchor)


   [modules/Project/**SaveProject**](https://gitlab.com/remplanner/visual/-/blob/master/js/3d/src/modules/Project/ProjectSave.js?ref_type=heads)
   - создание объекта текущих изменений проекта пользователя и отправка на сервер


[modules/**Mode**](https://gitlab.com/remplanner/visual/-/blob/master/js/3d/src/modules/Mode.js?ref_type=heads)
- создается точкой входа Remplanner3D
- текущий режим взаимодействия с пользователем (Обычный вид, Ходьба, Скрытие стен)
- сохраняет пользовательские ракурсы камер в LocalStorage и подтягивает оттуда


[modules/**Decor**](https://gitlab.com/remplanner/visual/-/blob/master/js/3d/src/modules/Decor/Decor.js?ref_type=heads)
- создается точкой входа Remplanner3D
- создает и при помощи [Engine/syatems/**Graphics**](https://gitlab.com/remplanner/visual/-/blob/master/js/3d/src/Engine/systems/Graphics/Graphics.js?ref_type=heads) и добавляет в сцену меши декора


   [modules/Decor/**Placement**](https://gitlab.com/remplanner/visual/-/blob/master/js/3d/src/modules/Decor/Placement/Placement.js?ref_type=heads)
   - отвечает за перемещение декора


   [modules/Decor/**Lair**](https://gitlab.com/remplanner/visual/-/blob/master/js/3d/src/modules/Decor/Lair.js?ref_type=heads)
    - отвечает за поверхности для размещения декора


[modules/**Skin**](https://gitlab.com/remplanner/visual/-/blob/master/js/3d/src/modules/Skin/Skin.js?ref_type=heads)
- создается точкой входа Remplanner3D
- из проекта пользователя сигналит в сцену создать материалы
- `#create(options)` ищет материал в сохраненном проекте пользователя, если находит то берет дефолтный материал из [modules/**Registry**](https://gitlab.com/remplanner/visual/-/blob/master/js/3d/src/modules/Registry.js?ref_type=heads) и применяет к нему cохраненные настройки, если не находит то берет материал из [modules/Skin/**Presets**](https://gitlab.com/remplanner/visual/-/blob/master/js/3d/src/modules/Skin/Presets.js?ref_type=heads) и сохраняет в проекте кастомные настройки.
	- `options.saveKey` - ключ по которому материал ищется в сохраненном проекте пользователя и сохраняется если не находится и возвращается
	- `options.presetId` - если options.saveKey не находит материал то материал берется из [modules/Skin/**Presets**](https://gitlab.com/remplanner/visual/-/blob/master/js/3d/src/modules/Skin/Presets.js?ref_type=heads)


[modules/**Entourage**](https://gitlab.com/remplanner/visual/-/blob/master/js/3d/src/modules/Entourage/Entourage.js?ref_type=heads)
- создается точкой входа Remplanner3D
- создает студию для рендера


[modules/**Spatium**](https://gitlab.com/remplanner/visual/-/blob/master/js/3d/src/modules/Spatium/Spatium.js?ref_type=heads)
- создается точкой входа Remplanner3D
- рендер в картинку
- выгрузка модели квартиры .glb


[modules/**Apartments**](https://gitlab.com/remplanner/visual/-/blob/master/js/3d/src/modules/Apartments.js?ref_type=heads)
- создается точкой входа Remplanner3D
- создает коробку квартиры
- создает экземпляры modules/Walls, modules/Ceiling, modules/Portal, modules/Interior, modules/Floor, modules/PlinthsFloor, modules/PlinthsCeiling


**3.3 Modules создаваемые в `modules/Apartments`**

[modules/**Wall**](https://gitlab.com/remplanner/visual/-/blob/master/js/3d/src/modules/Wall/Wall.js?ref_type=heads)
- создается в modules/Apartments
- хранит данные о стенах
- создает экземпляры следующих классов


	[modules/Wall/**WallNiches**](https://gitlab.com/remplanner/visual/-/blob/master/js/3d/src/modules/Wall/WallNiches.js?ref_type=heads)
	- создает ниши
	- хранит данные о нишах


	[modules/Wall/**WallMoldings**](https://gitlab.com/remplanner/visual/-/blob/master/js/3d/src/modules/Wall/WallMoldings.js?ref_type=heads)
	- создает молдинги
	- хранит данные о молдингах


 - пользуется следующими методами


   	[modules/Wall/**WallFaces**](https://gitlab.com/remplanner/visual/-/blob/master/js/3d/src/modules/Wall/WallFaces.js?ref_type=heads)
   	- создает, добавляет стены

  
   	[modules/Wall/**WallCaps**](https://gitlab.com/remplanner/visual/-/blob/master/js/3d/src/modules/Wall/WallCaps.js?ref_type=heads)
   	- создает, добавляет верх стен


	[modules/Wall/Generators/**WallGlass**](https://gitlab.com/remplanner/visual/-/blob/master/js/3d/src/modules/Wall/Generators/WallGlass.js?ref_type=heads)
	- создает, добавляет стеклянные стены


	[modules/Wall/**WallGeometry**](https://gitlab.com/remplanner/visual/-/blob/master/js/3d/src/modules/Wall/WallGeometry.js?ref_type=heads)
	- Набор вспомогательных методов создания геометрии для всех классов стен


[modules/**Portal**](https://gitlab.com/remplanner/visual/-/blob/master/js/3d/src/modules/Portal/Portal.js?ref_type=heads)
- создается в modules/Apartments
- Создает внутренние откосы, окна, двери
- пользуется методами modules/Wall/WallFaces, modules/Wall/WallGeometries
- добавляет в сцену пустые проемы, двери и окна


	[modules/Portal/**PortalFrame**](https://gitlab.com/remplanner/visual/-/blob/master/js/3d/src/modules/Portal/PortalFrame.js?ref_type=heads)
	- хранит данные о порталах
	- набор статических методов создания данных для Окон и Дверей


	[modules/Portal/**PortalSlopes**](https://gitlab.com/remplanner/visual/-/blob/master/js/3d/src/modules/Portal/PortalSlopes.js?ref_type=heads)
	- набор статических методов создания откосов


[modules/**Ceiling**](https://gitlab.com/remplanner/visual/-/blob/master/js/3d/src/modules/Ceiling/Ceiling.js?ref_type=heads)
- создается в modules/Apartments
- создает потолки
- хранит данные о потолках


**3.4 Modules статические, нигде не создаются работают без контекста**


[modules/**Model**](https://gitlab.com/remplanner/visual/-/blob/master/js/3d/src/modules/Model/Model.js?ref_type=heads)
- создатель мешей
- `.load()` запрашивает в [modules/**Registry**](https://gitlab.com/remplanner/visual/-/blob/master/js/3d/src/modules/Registry.js?ref_type=heads) данные материала, подгружает меш, добавляет в сцену


**3.5 Systems живут изолированно, общаются через медиатор, создаются из remplanner-cfg в Engine**


[systems/**UserInterface**](https://gitlab.com/remplanner/visual/-/blob/master/js/3d/src/systems/UserInterface/UserInterface.js?ref_type=heads) <a id="user-interface-anchor">
- инитится через remplanner-cfg кнопочная обвязка
- создает боковые панели [systems/UserInterface/sections](https://gitlab.com/remplanner/visual/-/tree/master/js/3d/src/systems/UserInterface/sections?ref_type=heads)
- создает миникарту
….
- через [systems/UserInterface/**InterfaceActions**](https://gitlab.com/remplanner/visual/-/blob/master/js/3d/src/systems/UserInterface/InterfaceActions.js?ref_type=heads) сигналит медиатору о событиях UI


[systems/**Debug**](https://gitlab.com/remplanner/visual/-/blob/master/js/3d/src/systems/Debug/Debug.js?ref_type=heads)
- инитится через remplanner-cfg
- методы и ui дебага


[systems/**Walker**](https://gitlab.com/remplanner/visual/-/blob/master/js/3d/src/systems/Walker/Walker.js?ref_type=heads)
- инитится через remplanner-cfg
- активирует через медиатор Engine/systems/NavigatorFPerson
- хранит данные о порталах коллизиях в квартире


[systems/**Selector**](https://gitlab.com/remplanner/visual/-/blob/master/js/3d/src/systems/Selector/Selector.js?ref_type=heads)
- инитится через remplanner-cfg
- хранит текущий фокус на элементе
- выбирает элемент если в его меше свойства `userData.fixed = false`


[systems/**BareRoom**](https://gitlab.com/remplanner/visual/-/blob/master/js/3d/src/systems/BareRoom/BareRoom.js?ref_type=heads)
- инитится через remplanner-cfg
- обрезает квартиру по высоте в режиме просмотра 1м


**3.6 Описание [Engine](https://gitlab.com/remplanner/visual/-/blob/master/js/3d/src/Engine/Engine.js?ref_type=heads)**


[Engine](https://gitlab.com/remplanner/visual/-/blob/master/js/3d/src/Engine/Engine.js?ref_type=heads)
- [описание_выше](#engine-anchor)


**3.7 Engine/systems экземпляры живут изолированно, общение-медиатор, создаются из remplanner-cfg в Engine**


[Engine/systems/**NavigatorFPerson**](https://gitlab.com/remplanner/visual/-/blob/master/js/3d/src/Engine/systems/NavigatorFPerson/NavigatorFPerson.js?ref_type=heads)
- инитится через remplanner-cfg
- слушает вводы пользователя
- двигает камеру


[Engine/systems/**Graphics**](https://gitlab.com/remplanner/visual/-/blob/master/js/3d/src/Engine/systems/Graphics/Graphics.js?ref_type=heads)
- инитится через remplanner-cfg
- подключает всю графику к медиатору


	[Engine/systems/Graphics/**SharedData**](https://gitlab.com/remplanner/visual/-/blob/master/js/3d/src/Engine/systems/Graphics/SharedData.js?ref_type=heads)
	- Статически хранит все меши в открытом доступе


	[Engine/systems/Graphics/**Meshes**](https://gitlab.com/remplanner/visual/-/blob/master/js/3d/src/Engine/systems/Graphics/Meshes.js?ref_type=heads)
	- методы медиатора через Engine/systems/Graphics
	- `.create(options)` по данным создает геометрию в Graphics/Geometries, по данным создает материал в Graphics/Materials, создает меш, добавляет меш в сцену через Graphics/Entities, регистрирует в Graphics/SharedData
		- `options.userData.fixed = true` меш не чекается селектором
		- `options.userData.rayObstruct = true` сквозь меш нельзя выбрать предмет
		- `options.userData.entity.roomId = ‘room_id_mesh_in’` скрывает меш при покомнатном просмотре не принадлежащий текущей комнате
		- `options.name = ‘mesh_unique_name’` служит для применения сохраненного пользователем материала при перезагрузке
		- `options.material.saveKey = [‘mesh_unique_name’, ‘mesh_unique_name_2’]` применяет при перезагрузке материалы к мешам по имени
		- `options.material.presetId = ‘1262’` значение это ключ из хранилища Registry.#materials
		- `options.material.group = ‘group_unique_name’` при изменении материала у одного члена группы с этим айди, меняется материал у всех элементов с этим айди в свойстве группы материала


[Engine/systems/Graphics/**SceneBasics**](https://gitlab.com/remplanner/visual/-/blob/master/js/3d/src/Engine/systems/Graphics/SceneBasics.js?ref_type=heads)
- методы медиатора через Engine/systems/Graphics
- Набор методов для создания и добавления в сцену вспомогательных объектов (Groups, Helpers, Lights..)


[Engine/systems/Graphics/**Entities**](https://gitlab.com/remplanner/visual/-/blob/master/js/3d/src/Engine/systems/Graphics/Entities.js?ref_type=heads)
- методы медиатора через Engine/systems/Graphics
- Набор методов для трансформации мешей, замены их геометрии и материалов, добавления к родителю. Каждый метод получает данные, берет меш из Graphics/SharedData и изменяет его.


[Engine/systems/Graphics/Geometries](https://gitlab.com/remplanner/visual/-/blob/master/js/3d/src/Engine/systems/Graphics/Geometries.js?ref_type=heads)
- методы медиатора через Engine/systems/Graphics
- Набор методов для создания и модификации геометрии
- `setProps(options)` изменени\добавление свойств к геометрии
	- `options = { uvApplyPlane: { size: 100, offsetV: 15 } }` - если все вершины геометрии находятся в одной плоскости в любом положении в пространстве к ним добавляется координаты размещения текстуры начиная с нижнего левого края и высотой и шириной  текстуры 100 и отступом снизу 15


[Engine/systems/Graphics/**Materials**](https://gitlab.com/remplanner/visual/-/blob/master/js/3d/src/Engine/systems/Graphics/Materials.js?ref_type=heads)
- методы медиатора через Engine/systems/Graphics
- Набор методов для создания и модификации материалов


[Engine/systems/Graphics/**Raycast**](https://gitlab.com/remplanner/visual/-/blob/master/js/3d/src/Engine/systems/Graphics/Raycast.js?ref_type=heads)
- методы медиатора через Engine/systems/Graphics
- Набор методов для выбора 3д модели под мышью


**3.8 systems/UserInterface более подробное**


[systems/**UserInterface**](https://gitlab.com/remplanner/visual/-/blob/master/js/3d/src/systems/UserInterface/UserInterface.js?ref_type=heads)
- [описание выше](#user-interface-anchor)


	[systems/UserInterface/**Elements**](https://gitlab.com/remplanner/visual/-/blob/master/js/3d/src/systems/UserInterface/Elements.js?ref_type=heads)
	- Набор методов для модификации html, общие для всего UserInterface



## 4. Аббревиатуры событий медиатора

| Сокращение | Значение |
|------------|----------|
| `gr` | Graphics |
| `grEnt` | Graphics/Entities |
| `plcDecor` | Decor/Placement |
| `regGet` | Запрос к Registry |
| `prj` | Действие Project |
| `prjGet` | Запрос к Project |
| `dbgControl` | systems/Debug (пока не используется) |
| `sel` | systems/Selector |


## 5. Описание данных приходящих с сервера

`.items[]` - данные о элементах декора, светильниках, трубах раскиданных по квартире
`.walls[]` - данные о стенах  

[Project/**OldVersion**](https://gitlab.com/remplanner/visual/-/blob/master/js/3d/src/modules/Project/OldVersion.js?ref_type=heads)<a id="oldversion-anchor">
- `rectifyProjectV3` 
	- собирает отдельный массив `portals` из исходных данных `walls[].holes`
	- добавляет к `portals[].type` `glass_` если стена к которой принадлежит портал стеклянная

---

© Remplanner3D