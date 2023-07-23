wait()

local Player = game.Players.LocalPlayer

--// Services \\--

local InputService = game:GetService("UserInputService")
local GuiService = game:GetService("GuiService")
local MarketService = game:GetService("MarketplaceService")

--// Modules \\--

local ItemModule = require(game.ReplicatedStorage.Modules.Items)
local DrawInvModule = require(game.ReplicatedStorage.Modules.DrawInv)
local CollectionModule = require(game.ReplicatedStorage.Modules.Collections)

--// Radio Game Pass \\--

local RadioID = 95529181
local PlayerOwnsRadio = MarketService:UserOwnsGamePassAsync(Player.UserId, RadioID)

--// Variables \\--

-- Sets up some variables and paths
local ImageString = "rbxgameasset://Images/"
local ButtonColor = game.ReplicatedStorage.ButtonColor.Value
local Remotes = game.ReplicatedStorage.Remotes

--// UI's \\--
-- Gets references to different UI elements
local sidebar = script.Parent.Parent.Sidebar.Inventory
local Inventory = script.Parent
local KnifeInv = Inventory.KnifeInv
local EffectInv = Inventory.EffectsInv
local RadioInv = Inventory.RadiosInv
local SideBarKnife = Inventory.Sidebar.Knife
local SideBarEffect = Inventory.Sidebar.Effects
local SideBarRadio = Inventory.Sidebar.Radios
local PageButtons = Inventory.Header.SectionButtons.Buttons
local Header = Inventory.Header

-- Set up the player's name and icon on da ui
InputService.ModalEnabled = true
InputService.ModalEnabled = false
Header.TextLabel.Text = Player.Name.. "'s Inventory"
Header.PlayerIcon.Image = "https://www.roblox.com/headshot-thumbnail/image?userId="..Player.UserId.."&width=420&height=420&format=png"

-- Function to set up the inventory ui with equipped items and available items
local SetupInv = function(EquippedKnife, EquippedEffect, EquippedRadio, InvKnife, InvEffect, InvRadio)
	-- Set the image of the equipped knife in the sidebar
	if EquippedKnife ~= nil then
		SideBarKnife.ItemImg.Image = ItemModule.GetImage(EquippedKnife)
		SideBarKnife.ItemImg.ImageLabel.Image = ItemModule.GetImage(EquippedKnife)
	end

	-- Set the image of the equipped effect in the sidebar
	if EquippedEffect ~= nil then
		SideBarEffect.ItemImg.Image = tostring(ImageString .. EquippedEffect)
		SideBarEffect.ItemImg.ImageLabel.Image = tostring(ImageString .. EquippedEffect)
	end

	-- Set the image of the equipped radio in the sidebar
	if EquippedRadio ~= nil then
		SideBarRadio.ItemImg.Image = tostring(ImageString .. EquippedRadio)
		SideBarRadio.ItemImg.ImageLabel.Image = tostring(ImageString .. EquippedRadio)
	end

	-- Clear the grids for knives, effects, and radios in the inventory
	CollectionModule.ClearGrid(KnifeInv.KnifeGrid, "ImageButton")
	CollectionModule.ClearGrid(EffectInv.EffectsGrid, "ImageButton")
	CollectionModule.ClearGrid(RadioInv.RadiosGrid, "ImageButton")

	-- Get and draw the inventory items for knives, effects, and radios
	InvKnife = DrawInvModule.GetAutoSort(InvKnife)
	DrawInvModule.Draw(KnifeInv.KnifeGrid, InvKnife, KnifeInv, KnifeInv.KnifeGrid.UIGridLayout)
	DrawInvModule.DrawNoMult(EffectInv.EffectsGrid, InvEffect, EffectInv, EffectInv.EffectsGrid.UIGridLayout)
	DrawInvModule.DrawNoMult(RadioInv.RadiosGrid, InvRadio, RadioInv, RadioInv.RadiosGrid.UIGridLayout)

	-- Connect click events for equipped items to equip them when clicked
	for _, i in pairs(KnifeInv.KnifeGrid:GetChildren()) do
		if i:IsA("ImageButton") then
			i.MouseButton1Click:Connect(CollectionModule.Debounce(function()
				SideBarKnife.ItemImg.Image = ItemModule.GetImage(i.Name)
				SideBarKnife.ItemImg.ImageLabel.Image = ItemModule.GetImage(i.Name)
				Remotes.EquipKnife:FireServer(i.Name)
			end))
		end
	end

	for _, i in pairs(EffectInv.EffectsGrid:GetChildren()) do
		if i:IsA("ImageButton") then
			i.MouseButton1Click:Connect(CollectionModule.Debounce(function()
				SideBarEffect.ItemImg.Image = ItemModule.GetImage(i.Name)
				SideBarEffect.ItemImg.ImageLabel.Image = ItemModule.GetImage(i.Name)
				Remotes.EquipEffect:FireServer(i.Name)
			end))
		end
	end

	-- If the player owns the radio game pass, connect click events for radios to equip them when clicked
	if PlayerOwnsRadio then
		RadioInv.LockedFrame.Visible = false
		for _, i in pairs(RadioInv.RadiosGrid:GetChildren()) do
			if i:IsA("ImageButton") then
				i.MouseButton1Click:Connect(CollectionModule.Debounce(function()
					SideBarRadio.ItemImg.Image = ItemModule.GetImage(i.Name)
					SideBarRadio.ItemImg.ImageLabel.Image = ItemModule.GetImage(i.Name)
					Remotes.EquipRadio:FireServer(i.Name)
				end))
			end
		end
	else
		-- If the player doesn't own the radio game pass, show the locked frame
		RadioInv.LockedFrame.Visible = true
	end
end

-- Connect to the inventory update event and call SetupInv to set up the inventory
Remotes.InvUpd.OnClientEvent:Connect(function(a, b, c, d, e, f, g, h)
	SetupInv(a, b, c, d, e, f, g, h)
	wait()
end)

-- Set the default inventory to "Knife" and hide other inventories
local CurrentInv = "Knife"
KnifeInv.Visible = true
EffectInv.Visible = false
RadioInv.Visible = false

-- Connect click events for switching between inventory categories (Knives, Effects, Radios)
PageButtons.Knives.MouseButton1Click:Connect(CollectionModule.Debounce(function()
	CurrentInv = "Knife"
	SideBarKnife.Visible = true
	SideBarEffect.Visible = false
	SideBarRadio.Visible = false

	Inventory.Sidebar.CraftingButton.Visible = true

	KnifeInv.Visible = true
	EffectInv.Visible = false
	RadioInv.Visible = false
	Inventory.SearchBar.Visible = true
	PageButtons.Knives.BackgroundColor3 = Color3.fromRGB(85, 170, 127)
	PageButtons.Effects.BackgroundColor3 =  Color3.fromRGB(22, 22, 22)
	PageButtons.Radios.BackgroundColor3 =  Color3.fromRGB(22, 22, 22)
end))

PageButtons.Effects.MouseButton1Click:Connect(CollectionModule.Debounce(function()
	CurrentInv = "Effects"
	SideBarKnife.Visible = false
	SideBarEffect.Visible = true
	SideBarRadio.Visible = false

	Inventory.Sidebar.CraftingButton.Visible = false

	KnifeInv.Visible = false
	EffectInv.Visible = true
	RadioInv.Visible = false

	Inventory.SearchBar.Visible = false
	PageButtons.Effects.BackgroundColor3 = Color3.fromRGB(85, 170, 127)
	PageButtons.Knives.BackgroundColor3 =  Color3.fromRGB(22, 22, 22)
	PageButtons.Radios.BackgroundColor3 =  Color3.fromRGB(22, 22, 22)
end))

PageButtons.Radios.MouseButton1Click:Connect(CollectionModule.Debounce(function()
	CurrentInv = "Radios"
	SideBarKnife.Visible = false
	SideBarEffect.Visible = false
	SideBarRadio.Visible = true

	Inventory.Sidebar.CraftingButton.Visible = false

	KnifeInv.Visible = false
	EffectInv.Visible = false
	RadioInv.Visible = true

	Inventory.SearchBar.Visible = false
	PageButtons.Radios.BackgroundColor3 = Color3.fromRGB(85, 170, 127)
	PageButtons.Effects.BackgroundColor3 =  Color3.fromRGB(22, 22, 22)
	PageButtons.Knives.BackgroundColor3 =  Color3.fromRGB(22, 22, 22)
end))

-- Connect click events for the sidebar items (equipped items) to equip the default items when clicked
SideBarKnife.MouseButton1Click:Connect(CollectionModule.Debounce(function()
	SideBarKnife.ItemImg.Image = ItemModule.GetImage("Basic")
	SideBarKnife.ItemImg.ImageLabel.Image = ItemModule.GetImage("Basic")
	Remotes.EquipKnife:FireServer("Basic")
end))

SideBarEffect.MouseButton1Click:Connect(CollectionModule.Debounce(function()
	SideBarEffect.ItemImg.Image = tostring(ImageString .. "")
	SideBarEffect.ItemImg.ImageLabel.Image = tostring(ImageString .. "")
	Remotes.EquipEffect:FireServer("")
end))

-- If the player owns the radio game pass, connect the click event for the radio in the sidebar
if PlayerOwnsRadio then
	SideBarRadio.MouseButton1Click:Connect(CollectionModule.Debounce(function()
		if PlayerOwnsRadio then
			SideBarRadio.ItemImg.Image = tostring(ImageString .. "")
			SideBarRadio.ItemImg.ImageLabel.Image = tostring(ImageString .. "")
			Remotes.EquipRadio:FireServer("")
		end
	end))
end

-- Check if the sidebar radio has an image and set a default image if not
if SideBarRadio.ItemImg.Image == "" then
	SideBarRadio.ItemImg.Image = ItemModule.GetImage("Normal")
	SideBarRadio.ItemImg.ImageLabel.Image = ItemModule.GetImage("Normal")
end

-- Connect a click event for the sidebar to update the canvas size of the knife inventory grid
local CurrentProcess = game:GetService("HttpService"):GenerateGUID()

sidebar.Parent.Inventory.MouseButton1Click:Connect(function()
	KnifeInv.CanvasSize = UDim2.new(0, 0, 0, KnifeInv.KnifeGrid.UIGridLayout.AbsoluteContentSize.Y + 10)
end)

-- Function to handle search when the player types in the search bar
local SearchBar = Inventory.SearchBar
local CurrentProcess = game:GetService("HttpService"):GenerateGUID()
function Search()
	local Text = SearchBar.SearchText.Text
	local ProcessId = game:GetService("HttpService"):GenerateGUID()
	CurrentProcess = ProcessId
	if Text then
		for i, v in next, KnifeInv.KnifeGrid:GetChildren() do
			if CurrentProcess ~= ProcessId then return end
			if v:IsA("ImageButton") then
				v.Visible = (v.Name:lower():find(Text:lower()))
			end
		end
		KnifeInv.CanvasSize = UDim2.new(0, 0, 0, KnifeInv.KnifeGrid.UIGridLayout.AbsoluteContentSize.Y + 10)	
	end
end

-- Connect the search function to the focus lost event of the search bar
SearchBar.SearchText.FocusLost:Connect(function(enter)
	if enter then
		Search()
	end
end)

-- Variable to prevent multiple execution of the character added event handler
local joined = false

-- Connect the character added event to update the inventory UI
Player.CharacterAdded:Connect(function(character)
	if joined then return end
	KnifeInv.CanvasSize = UDim2.new(0, 0, 0, KnifeInv.KnifeGrid.UIGridLayout.AbsoluteContentSize.Y + 10)
	KnifeInv.CanvasSize = UDim2.new(0, 0, 0, KnifeInv.KnifeGrid.UIGridLayout.AbsoluteContentSize.Y + 10)	
	joined = true
end)

-- Connect click events for closing the inventory and opening the crafting page
Inventory.Close.MouseButton1Click:Connect(CollectionModule.Debounce(function()
	Inventory.Visible = false
end))

Inventory.Sidebar.CraftingButton.MouseButton1Click:Connect(CollectionModule.Debounce(function()
	Inventory.Visible = false
	Inventory.Parent.CraftPage.Visible = true
end))
