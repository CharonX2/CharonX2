-- init
local player = game.Players.LocalPlayer
local mouse = playerGetMouse()

-- services
local input = gameGetService(UserInputService)
local run = gameGetService(RunService)
local tween = gameGetService(TweenService)
local tweeninfo = TweenInfo.new

-- additional
local utility = {}

-- themes
local objects = {}
local themes = {
	Background = Color3.fromRGB(15,15,15), 
	Glow = Color3.fromRGB(0, 0, 0), 
	Accent = Color3.fromRGB(10, 10, 10), 
	LightContrast = Color3.fromRGB(20, 20, 20), 
	DarkContrast = Color3.fromRGB(10,10,10),  
	TextColor = Color3.fromRGB(127, 0, 255)
}

do
	function utilityCreate(instance, properties, children)
		local object = Instance.new(instance)

		for i, v in pairs(properties or {}) do
			object[i] = v

			if typeof(v) == Color3 then -- save for theme changer later
				local theme = utilityFind(themes, v)

				if theme then
					objects[theme] = objects[theme] or {}
					objects[theme][i] = objects[theme][i] or setmetatable({}, {_mode = k})

					table.insert(objects[theme][i], object)
				end
			end
		end

		for i, module in pairs(children or {}) do
			module.Parent = object
		end

		return object
	end

	function utilityTween(instance, properties, duration, ...)
		tweenCreate(instance, tweeninfo(duration, ...), properties)Play()
	end

	function utilityWait()
		run.RenderSteppedWait()
		return true
	end

	function utilityFind(table, value) -- table.find doesn't work for dictionaries
		for i, v in  pairs(table) do
			if v == value then
				return i
			end
		end
	end

	function utilitySort(pattern, values)
		local new = {}
		pattern = patternlower()

		if pattern ==  then
			return values
		end

		for i, value in pairs(values) do
			if tostring(value)lower()find(pattern) then
				table.insert(new, value)
			end
		end

		return new
	end

	function utilityPop(object, shrink)
		local clone = objectClone()

		clone.AnchorPoint = Vector2.new(0.5, 0.5)
		clone.Size = clone.Size - UDim2.new(0, shrink, 0, shrink)
		clone.Position = UDim2.new(0.5, 0, 0.5, 0)

		clone.Parent = object
		cloneClearAllChildren()

		object.ImageTransparency = 1
		utilityTween(clone, {Size = object.Size}, 0.2)

		spawn(function()
			wait(0.2)

			object.ImageTransparency = 0
			cloneDestroy()
		end)

		return clone
	end

	function utilityInitializeKeybind()
		self.keybinds = {}
		self.ended = {}

		input.InputBeganConnect(function(key)
			if self.keybinds[key.KeyCode] then
				for i, bind in pairs(self.keybinds[key.KeyCode]) do
					bind()
				end
			end
		end)

		input.InputEndedConnect(function(key)
			if key.UserInputType == Enum.UserInputType.MouseButton1 then
				for i, callback in pairs(self.ended) do
					callback()
				end
			end
		end)
	end

	function utilityBindToKey(key, callback)

		self.keybinds[key] = self.keybinds[key] or {}

		table.insert(self.keybinds[key], callback)

		return {
			UnBind = function()
				for i, bind in pairs(self.keybinds[key]) do
					if bind == callback then
						table.remove(self.keybinds[key], i)
					end
				end
			end
		}
	end

	function utilityKeyPressed() -- yield until next key is pressed
		local key = input.InputBeganWait()

		while key.UserInputType ~= Enum.UserInputType.Keyboard	 do
			key = input.InputBeganWait()
		end

		wait() -- overlapping connection

		return key
	end

	function utilityDraggingEnabled(frame, parent)

		parent = parent or frame

		-- stolen from wally or kiriot, kek
		local dragging = false
		local dragInput, mousePos, framePos

		frame.InputBeganConnect(function(input)
			if input.UserInputType == Enum.UserInputType.MouseButton1 then
				dragging = true
				mousePos = input.Position
				framePos = parent.Position

				input.ChangedConnect(function()
					if input.UserInputState == Enum.UserInputState.End then
						dragging = false
					end
				end)
			end
		end)

		frame.InputChangedConnect(function(input)
			if input.UserInputType == Enum.UserInputType.MouseMovement then
				dragInput = input
			end
		end)

		input.InputChangedConnect(function(input)
			if input == dragInput and dragging then
				local delta = input.Position - mousePos
				parent.Position  = UDim2.new(framePos.X.Scale, framePos.X.Offset + delta.X, framePos.Y.Scale, framePos.Y.Offset + delta.Y)
			end
		end)

	end

	function utilityDraggingEnded(callback)
		table.insert(self.ended, callback)
	end

end

-- classes

local library = {} -- main
local page = {}
local section = {}

do
	library.__index = library
	page.__index = page
	section.__index = section

	-- new classes

	function library.new(title)
		local container = utilityCreate(ScreenGui, {
			Name = title,
			Parent = game.CoreGui
		}, {
			utilityCreate(ImageLabel, {
				Name = Main,
				BackgroundTransparency = 1,
				Position = UDim2.new(0.25, 0, 0.052435593, 0),
				Size = UDim2.new(0, 511, 0, 428),
				Image = rbxassetid4641149554,
				ImageColor3 = themes.Background,
				ScaleType = Enum.ScaleType.Slice,
				SliceCenter = Rect.new(4, 4, 296, 296)
			}, {
				utilityCreate(ImageLabel, {
					Name = Glow,
					BackgroundTransparency = 1,
					Position = UDim2.new(0, -15, 0, -15),
					Size = UDim2.new(1, 30, 1, 30),
					ZIndex = 0,
					Image = rbxassetid5028857084,
					ImageColor3 = themes.Glow,
					ScaleType = Enum.ScaleType.Slice,
					SliceCenter = Rect.new(24, 24, 276, 276)
				}),
				utilityCreate(ImageLabel, {
					Name = Pages,
					BackgroundTransparency = 1,
					ClipsDescendants = true,
					Position = UDim2.new(0, 0, 0, 38),
					Size = UDim2.new(0, 126, 1, -38),
					ZIndex = 3,
					Image = rbxassetid5012534273,
					ImageColor3 = themes.DarkContrast,
					ScaleType = Enum.ScaleType.Slice,
					SliceCenter = Rect.new(4, 4, 296, 296)
				}, {
					utilityCreate(ScrollingFrame, {
						Name = Pages_Container,
						Active = true,
						BackgroundTransparency = 1,
						Position = UDim2.new(0, 0, 0, 10),
						Size = UDim2.new(1, 0, 1, -20),
						CanvasSize = UDim2.new(0, 0, 0, 314),
						ScrollBarThickness = 0
					}, {
						utilityCreate(UIListLayout, {
							SortOrder = Enum.SortOrder.LayoutOrder,
							Padding = UDim.new(0, 10)
						})
					})
				}),
				utilityCreate(ImageLabel, {
					Name = TopBar,
					BackgroundTransparency = 1,
					ClipsDescendants = true,
					Size = UDim2.new(1, 0, 0, 38),
					ZIndex = 5,
					Image = rbxassetid4595286933,
					ImageColor3 = themes.Accent,
					ScaleType = Enum.ScaleType.Slice,
					SliceCenter = Rect.new(4, 4, 296, 296)
				}, {
					utilityCreate(TextLabel, { -- title
						Name = Title,
						AnchorPoint = Vector2.new(0, 0.5),
						BackgroundTransparency = 1,
						Position = UDim2.new(0, 12, 0, 19),
						Size = UDim2.new(1, -46, 0, 16),
						ZIndex = 5,
						Font = Enum.Font.GothamBold,
						Text = title,
						TextColor3 = themes.TextColor,
						TextSize = 14,
						TextXAlignment = Enum.TextXAlignment.Left
					})
				})
			})
		})

		utilityInitializeKeybind()
		utilityDraggingEnabled(container.Main.TopBar, container.Main)

		return setmetatable({
			container = container,
			pagesContainer = container.Main.Pages.Pages_Container,
			pages = {}
		}, library)
	end

	function page.new(library, title, icon)
		local button = utilityCreate(TextButton, {
			Name = title,
			Parent = library.pagesContainer,
			BackgroundTransparency = 1,
			BorderSizePixel = 0,
			Size = UDim2.new(1, 0, 0, 26),
			ZIndex = 3,
			AutoButtonColor = false,
			Font = Enum.Font.Gotham,
			Text = ,
			TextSize = 14
		}, {
			utilityCreate(TextLabel, {
				Name = Title,
				AnchorPoint = Vector2.new(0, 0.5),
				BackgroundTransparency = 1,
				Position = UDim2.new(0, 40, 0.5, 0),
				Size = UDim2.new(0, 76, 1, 0),
				ZIndex = 3,
				Font = Enum.Font.Gotham,
				Text = title,
				TextColor3 = themes.TextColor,
				TextSize = 12,
				TextTransparency = 0.65,
				TextXAlignment = Enum.TextXAlignment.Left
			}),
			icon and utilityCreate(ImageLabel, {
				Name = Icon, 
				AnchorPoint = Vector2.new(0, 0.5),
				BackgroundTransparency = 1,
				Position = UDim2.new(0, 12, 0.5, 0),
				Size = UDim2.new(0, 16, 0, 16),
				ZIndex = 3,
				Image = rbxassetid .. tostring(icon),
				ImageColor3 = themes.TextColor,
				ImageTransparency = 0.64
			}) or {}
		})

		local container = utilityCreate(ScrollingFrame, {
			Name = title,
			Parent = library.container.Main,
			Active = true,
			BackgroundTransparency = 1,
			BorderSizePixel = 0,
			Position = UDim2.new(0, 134, 0, 46),
			Size = UDim2.new(1, -142, 1, -56),
			CanvasSize = UDim2.new(0, 0, 0, 466),
			ScrollBarThickness = 3,
			ScrollBarImageColor3 = themes.DarkContrast,
			Visible = false
		}, {
			utilityCreate(UIListLayout, {
				SortOrder = Enum.SortOrder.LayoutOrder,
				Padding = UDim.new(0, 10)
			})
		})

		return setmetatable({
			library = library,
			container = container,
			button = button,
			sections = {}
		}, page)
	end

	function section.new(page, title)
		local container = utilityCreate(ImageLabel, {
			Name = title,
			Parent = page.container,
			BackgroundTransparency = 1,
			Size = UDim2.new(1, -10, 0, 28),
			ZIndex = 2,
			Image = rbxassetid5028857472,
			ImageColor3 = themes.LightContrast,
			ScaleType = Enum.ScaleType.Slice,
			SliceCenter = Rect.new(4, 4, 296, 296),
			ClipsDescendants = true
		}, {
			utilityCreate(Frame, {
				Name = Container,
				Active = true,
				BackgroundTransparency = 1,
				BorderSizePixel = 0,
				Position = UDim2.new(0, 8, 0, 8),
				Size = UDim2.new(1, -16, 1, -16)
			}, {
				utilityCreate(TextLabel, {
					Name = Title,
					BackgroundTransparency = 1,
					Size = UDim2.new(1, 0, 0, 20),
					ZIndex = 2,
					Font = Enum.Font.GothamSemibold,
					Text = title,
					TextColor3 = themes.TextColor,
					TextSize = 12,
					TextXAlignment = Enum.TextXAlignment.Left,
					TextTransparency = 1
				}),
				utilityCreate(UIListLayout, {
					SortOrder = Enum.SortOrder.LayoutOrder,
					Padding = UDim.new(0, 4)
				})
			})
		})

		return setmetatable({
			page = page,
			container = container.Container,
			colorpickers = {},
			modules = {},
			binds = {},
			lists = {},
		}, section) 
	end

	function libraryaddPage(...)

		local page = page.new(self, ...)
		local button = page.button

		table.insert(self.pages, page)

		button.MouseButton1ClickConnect(function()
			selfSelectPage(page, true)
		end)

		return page
	end

	function pageaddSection(...)
		local section = section.new(self, ...)

		table.insert(self.sections, section)

		return section
	end

	-- functions

	function librarysetTheme(theme, color3)
		themes[theme] = color3

		for property, objects in pairs(objects[theme]) do
			for i, object in pairs(objects) do
				if not object.Parent or (object.Name == Button and object.Parent.Name == ColorPicker) then
					objects[i] = nil -- i can do this because weak tables D
				else
					object[property] = color3
				end
			end
		end
	end

	function librarytoggle()

		if self.toggling then
			return
		end

		self.toggling = true

		local container = self.container.Main
		local topbar = container.TopBar

		if self.position then
			utilityTween(container, {
				Size = UDim2.new(0, 511, 0, 428),
				Position = self.position
			}, 0.2)
			wait(0.2)

			utilityTween(topbar, {Size = UDim2.new(1, 0, 0, 38)}, 0.2)
			wait(0.2)

			container.ClipsDescendants = false
			self.position = nil
		else
			self.position = container.Position
			container.ClipsDescendants = true

			utilityTween(topbar, {Size = UDim2.new(1, 0, 1, 0)}, 0.2)
			wait(0.2)

			utilityTween(container, {
				Size = UDim2.new(0, 511, 0, 0),
				Position = self.position + UDim2.new(0, 0, 0, 428)
			}, 0.2)
			wait(0.2)
		end

		self.toggling = false
	end

	-- new modules

	function libraryNotify(title, text, callback)

		-- overwrite last notification
		if self.activeNotification then
			self.activeNotification = self.activeNotification()
		end

		-- standard create
		local notification = utilityCreate(ImageLabel, {
			Name = Notification,
			Parent = self.container,
			BackgroundTransparency = 1,
			Size = UDim2.new(0, 200, 0, 60),
			Image = rbxassetid5028857472,
			ImageColor3 = themes.Background,
			ScaleType = Enum.ScaleType.Slice,
			SliceCenter = Rect.new(4, 4, 296, 296),
			ZIndex = 3,
			ClipsDescendants = true
		}, {
			utilityCreate(ImageLabel, {
				Name = Flash,
				Size = UDim2.new(1, 0, 1, 0),
				BackgroundTransparency = 1,
				Image = rbxassetid4641149554,
				ImageColor3 = themes.TextColor,
				ZIndex = 5
			}),
			utilityCreate(ImageLabel, {
				Name = Glow,
				BackgroundTransparency = 1,
				Position = UDim2.new(0, -15, 0, -15),
				Size = UDim2.new(1, 30, 1, 30),
				ZIndex = 2,
				Image = rbxassetid5028857084,
				ImageColor3 = themes.Glow,
				ScaleType = Enum.ScaleType.Slice,
				SliceCenter = Rect.new(24, 24, 276, 276)
			}),
			utilityCreate(TextLabel, {
				Name = Title,
				BackgroundTransparency = 1,
				Position = UDim2.new(0, 10, 0, 8),
				Size = UDim2.new(1, -40, 0, 16),
				ZIndex = 4,
				Font = Enum.Font.GothamSemibold,
				TextColor3 = themes.TextColor,
				TextSize = 14.000,
				TextXAlignment = Enum.TextXAlignment.Left
			}),
			utilityCreate(TextLabel, {
				Name = Text,
				BackgroundTransparency = 1,
				Position = UDim2.new(0, 10, 1, -24),
				Size = UDim2.new(1, -40, 0, 16),
				ZIndex = 4,
				Font = Enum.Font.Gotham,
				TextColor3 = themes.TextColor,
				TextSize = 12.000,
				TextXAlignment = Enum.TextXAlignment.Left
			}),
			utilityCreate(ImageButton, {
				Name = Accept,
				BackgroundTransparency = 1,
				Position = UDim2.new(1, -26, 0, 8),
				Size = UDim2.new(0, 16, 0, 16),
				Image = rbxassetid5012538259,
				ImageColor3 = themes.TextColor,
				ZIndex = 4
			}),
			utilityCreate(ImageButton, {
				Name = Decline,
				BackgroundTransparency = 1,
				Position = UDim2.new(1, -26, 1, -24),
				Size = UDim2.new(0, 16, 0, 16),
				Image = rbxassetid5012538583,
				ImageColor3 = themes.TextColor,
				ZIndex = 4
			})
		})

		-- dragging
		utilityDraggingEnabled(notification)

		-- position and size
		title = title or Notification
		text = text or 

		notification.Title.Text = title
		notification.Text.Text = text

		local padding = 10
		local textSize = gameGetService(TextService)GetTextSize(text, 12, Enum.Font.Gotham, Vector2.new(math.huge, 16))

		notification.Position = library.lastNotification or UDim2.new(0, padding, 1, -(notification.AbsoluteSize.Y + padding))
		notification.Size = UDim2.new(0, 0, 0, 60)

		utilityTween(notification, {Size = UDim2.new(0, textSize.X + 70, 0, 60)}, 0.2)
		wait(0.2)

		notification.ClipsDescendants = false
		utilityTween(notification.Flash, {
			Size = UDim2.new(0, 0, 0, 60),
			Position = UDim2.new(1, 0, 0, 0)
		}, 0.2)

		-- callbacks
		local active = true
		local close = function()

			if not active then
				return
			end

			active = false
			notification.ClipsDescendants = true

			library.lastNotification = notification.Position
			notification.Flash.Position = UDim2.new(0, 0, 0, 0)
			utilityTween(notification.Flash, {Size = UDim2.new(1, 0, 1, 0)}, 0.2)

			wait(0.2)
			utilityTween(notification, {
				Size = UDim2.new(0, 0, 0, 60),
				Position = notification.Position + UDim2.new(0, textSize.X + 70, 0, 0)
			}, 0.2)

			wait(0.2)
			notificationDestroy()
		end

		self.activeNotification = close

		notification.Accept.MouseButton1ClickConnect(function()

			if not active then 
				return
			end

			if callback then
				callback(true)
			end

			close()
		end)

		notification.Decline.MouseButton1ClickConnect(function()

			if not active then 
				return
			end

			if callback then
				callback(false)
			end

			close()
		end)
	end

	function sectionaddButton(title, callback)
		local button = utilityCreate(ImageButton, {
			Name = Button,
			Parent = self.container,
			BackgroundTransparency = 1,
			BorderSizePixel = 0,
			Size = UDim2.new(1, 0, 0, 30),
			ZIndex = 2,
			Image = rbxassetid5028857472,
			ImageColor3 = themes.DarkContrast,
			ScaleType = Enum.ScaleType.Slice,
			SliceCenter = Rect.new(2, 2, 298, 298)
		}, {
			utilityCreate(TextLabel, {
				Name = Title,
				BackgroundTransparency = 1,
				Size = UDim2.new(1, 0, 1, 0),
				ZIndex = 3,
				Font = Enum.Font.Gotham,
				Text = title,
				TextColor3 = themes.TextColor,
				TextSize = 12,
				TextTransparency = 0.10000000149012
			})
		})

		table.insert(self.modules, button)
		--selfResize()

		local text = button.Title
		local debounce

		button.MouseButton1ClickConnect(function()

			if debounce then
				return
			end

			-- animation
			utilityPop(button, 10)

			debounce = true
			text.TextSize = 0
			utilityTween(button.Title, {TextSize = 14}, 0.2)

			wait(0.2)
			utilityTween(button.Title, {TextSize = 12}, 0.2)

			if callback then
				callback(function(...)
					selfupdateButton(button, ...)
				end)
			end

			debounce = false
		end)

		return button
	end

	function sectionaddToggle(title, default, callback)
		local toggle = utilityCreate(ImageButton, {
			Name = Toggle,
			Parent = self.container,
			BackgroundTransparency = 1,
			BorderSizePixel = 0,
			Size = UDim2.new(1, 0, 0, 30),
			ZIndex = 2,
			Image = rbxassetid5028857472,
			ImageColor3 = themes.DarkContrast,
			ScaleType = Enum.ScaleType.Slice,
			SliceCenter = Rect.new(2, 2, 298, 298)
		},{
			utilityCreate(TextLabel, {
				Name = Title,
				AnchorPoint = Vector2.new(0, 0.5),
				BackgroundTransparency = 1,
				Position = UDim2.new(0, 10, 0.5, 1),
				Size = UDim2.new(0.5, 0, 1, 0),
				ZIndex = 3,
				Font = Enum.Font.Gotham,
				Text = title,
				TextColor3 = themes.TextColor,
				TextSize = 12,
				TextTransparency = 0.10000000149012,
				TextXAlignment = Enum.TextXAlignment.Left
			}),
			utilityCreate(ImageLabel, {
				Name = Button,
				BackgroundTransparency = 1,
				BorderSizePixel = 0,
				Position = UDim2.new(1, -50, 0.5, -8),
				Size = UDim2.new(0, 40, 0, 16),
				ZIndex = 2,
				Image = rbxassetid5028857472,
				ImageColor3 = themes.LightContrast,
				ScaleType = Enum.ScaleType.Slice,
				SliceCenter = Rect.new(2, 2, 298, 298)
			}, {
				utilityCreate(ImageLabel, {
					Name = Frame,
					BackgroundTransparency = 1,
					Position = UDim2.new(0, 2, 0.5, -6),
					Size = UDim2.new(1, -22, 1, -4),
					ZIndex = 2,
					Image = rbxassetid5028857472,
					ImageColor3 = themes.TextColor,
					ScaleType = Enum.ScaleType.Slice,
					SliceCenter = Rect.new(2, 2, 298, 298)
				})
			})
		})

		table.insert(self.modules, toggle)
		--selfResize()

		local active = default
		local active = default
		selfupdateToggle(toggle, nil, active)

		toggle.MouseButton1ClickConnect(function()
			active = not active
			selfupdateToggle(toggle, nil, active)

			if callback then
				callback(active, function(...)
					selfupdateToggle(toggle, ...)
				end)
			end
		end)

		return toggle
	end

	function sectionaddTextbox(title, default, callback)
		local textbox = utilityCreate(ImageButton, {
			Name = Textbox,
			Parent = self.container,
			BackgroundTransparency = 1,
			BorderSizePixel = 0,
			Size = UDim2.new(1, 0, 0, 30),
			ZIndex = 2,
			Image = rbxassetid5028857472,
			ImageColor3 = themes.DarkContrast,
			ScaleType = Enum.ScaleType.Slice,
			SliceCenter = Rect.new(2, 2, 298, 298)
		}, {
			utilityCreate(TextLabel, {
				Name = Title,
				AnchorPoint = Vector2.new(0, 0.5),
				BackgroundTransparency = 1,
				Position = UDim2.new(0, 10, 0.5, 1),
				Size = UDim2.new(0.5, 0, 1, 0),
				ZIndex = 3,
				Font = Enum.Font.Gotham,
				Text = title,
				TextColor3 = themes.TextColor,
				TextSize = 12,
				TextTransparency = 0.10000000149012,
				TextXAlignment = Enum.TextXAlignment.Left
			}),
			utilityCreate(ImageLabel, {
				Name = Button,
				BackgroundTransparency = 1,
				Position = UDim2.new(1, -110, 0.5, -8),
				Size = UDim2.new(0, 100, 0, 16),
				ZIndex = 2,
				Image = rbxassetid5028857472,
				ImageColor3 = themes.LightContrast,
				ScaleType = Enum.ScaleType.Slice,
				SliceCenter = Rect.new(2, 2, 298, 298)
			}, {
				utilityCreate(TextBox, {
					Name = Textbox, 
					BackgroundTransparency = 1,
					TextTruncate = Enum.TextTruncate.AtEnd,
					Position = UDim2.new(0, 5, 0, 0),
					Size = UDim2.new(1, -10, 1, 0),
					ZIndex = 3,
					Font = Enum.Font.GothamSemibold,
					Text = default or ,
					TextColor3 = themes.TextColor,
					TextSize = 11
				})
			})
		})

		table.insert(self.modules, textbox)
		--selfResize()

		local button = textbox.Button
		local input = button.Textbox

		textbox.MouseButton1ClickConnect(function()

			if textbox.Button.Size ~= UDim2.new(0, 100, 0, 16) then
				return
			end

			utilityTween(textbox.Button, {
				Size = UDim2.new(0, 200, 0, 16),
				Position = UDim2.new(1, -210, 0.5, -8)
			}, 0.2)

			wait()

			input.TextXAlignment = Enum.TextXAlignment.Left
			inputCaptureFocus()
		end)

		inputGetPropertyChangedSignal(Text)Connect(function()

			if button.ImageTransparency == 0 and (button.Size == UDim2.new(0, 200, 0, 16) or button.Size == UDim2.new(0, 100, 0, 16)) then -- i know, i dont like this either
				utilityPop(button, 10)
			end

			if callback then
				callback(input.Text, nil, function(...)
					selfupdateTextbox(textbox, ...)
				end)
			end
		end)

		input.FocusLostConnect(function()

			input.TextXAlignment = Enum.TextXAlignment.Center

			utilityTween(textbox.Button, {
				Size = UDim2.new(0, 100, 0, 16),
				Position = UDim2.new(1, -110, 0.5, -8)
			}, 0.2)

			if callback then
				callback(input.Text, true, function(...)
					selfupdateTextbox(textbox, ...)
				end)
			end
		end)

		return textbox
	end

	function sectionaddKeybind(title, default, callback, changedCallback)
		local keybind = utilityCreate(ImageButton, {
			Name = Keybind,
			Parent = self.container,
			BackgroundTransparency = 1,
			BorderSizePixel = 0,
			Size = UDim2.new(1, 0, 0, 30),
			ZIndex = 2,
			Image = rbxassetid5028857472,
			ImageColor3 = themes.DarkContrast,
			ScaleType = Enum.ScaleType.Slice,
			SliceCenter = Rect.new(2, 2, 298, 298)
		}, {
			utilityCreate(TextLabel, {
				Name = Title,
				AnchorPoint = Vector2.new(0, 0.5),
				BackgroundTransparency = 1,
				Position = UDim2.new(0, 10, 0.5, 1),
				Size = UDim2.new(1, 0, 1, 0),
				ZIndex = 3,
				Font = Enum.Font.Gotham,
				Text = title,
				TextColor3 = themes.TextColor,
				TextSize = 12,
				TextTransparency = 0.10000000149012,
				TextXAlignment = Enum.TextXAlignment.Left
			}),
			utilityCreate(ImageLabel, {
				Name = Button,
				BackgroundTransparency = 1,
				Position = UDim2.new(1, -110, 0.5, -8),
				Size = UDim2.new(0, 100, 0, 16),
				ZIndex = 2,
				Image = rbxassetid5028857472,
				ImageColor3 = themes.LightContrast,
				ScaleType = Enum.ScaleType.Slice,
				SliceCenter = Rect.new(2, 2, 298, 298)
			}, {
				utilityCreate(TextLabel, {
					Name = Text,
					BackgroundTransparency = 1,
					ClipsDescendants = true,
					Size = UDim2.new(1, 0, 1, 0),
					ZIndex = 3,
					Font = Enum.Font.GothamSemibold,
					Text = default and default.Name or None,
					TextColor3 = themes.TextColor,
					TextSize = 11
				})
			})
		})

		table.insert(self.modules, keybind)
		--selfResize()

		local text = keybind.Button.Text
		local button = keybind.Button

		local animate = function()
			if button.ImageTransparency == 0 then
				utilityPop(button, 10)
			end
		end

		self.binds[keybind] = {callback = function()
			animate()

			if callback then
				callback(function(...)
					selfupdateKeybind(keybind, ...)
				end)
			end
		end}

		if default and callback then
			selfupdateKeybind(keybind, nil, default)
		end

		keybind.MouseButton1ClickConnect(function()

			animate()

			if self.binds[keybind].connection then -- unbind
				return selfupdateKeybind(keybind)
			end

			if text.Text == None then -- new bind
				text.Text = ...

				local key = utilityKeyPressed()

				selfupdateKeybind(keybind, nil, key.KeyCode)
				animate()

				if changedCallback then
					changedCallback(key, function(...)
						selfupdateKeybind(keybind, ...)
					end)
				end
			end
		end)

		return keybind
	end

	function sectionaddColorPicker(title, default, callback)
		local colorpicker = utilityCreate(ImageButton, {
			Name = ColorPicker,
			Parent = self.container,
			BackgroundTransparency = 1,
			BorderSizePixel = 0,
			Size = UDim2.new(1, 0, 0, 30),
			ZIndex = 2,
			Image = rbxassetid5028857472,
			ImageColor3 = themes.DarkContrast,
			ScaleType = Enum.ScaleType.Slice,
			SliceCenter = Rect.new(2, 2, 298, 298)
		},{
			utilityCreate(TextLabel, {
				Name = Title,
				AnchorPoint = Vector2.new(0, 0.5),
				BackgroundTransparency = 1,
				Position = UDim2.new(0, 10, 0.5, 1),
				Size = UDim2.new(0.5, 0, 1, 0),
				ZIndex = 3,
				Font = Enum.Font.Gotham,
				Text = title,
				TextColor3 = themes.TextColor,
				TextSize = 12,
				TextTransparency = 0.10000000149012,
				TextXAlignment = Enum.TextXAlignment.Left
			}),
			utilityCreate(ImageButton, {
				Name = Button,
				BackgroundTransparency = 1,
				BorderSizePixel = 0,
				Position = UDim2.new(1, -50, 0.5, -7),
				Size = UDim2.new(0, 40, 0, 14),
				ZIndex = 2,
				Image = rbxassetid5028857472,
				ImageColor3 = Color3.fromRGB(255, 255, 255),
				ScaleType = Enum.ScaleType.Slice,
				SliceCenter = Rect.new(2, 2, 298, 298)
			})
		})

		local tab = utilityCreate(ImageLabel, {
			Name = ColorPicker,
			Parent = self.page.library.container,
			BackgroundTransparency = 1,
			Position = UDim2.new(0.75, 0, 0.400000006, 0),
			Selectable = true,
			AnchorPoint = Vector2.new(0.5, 0.5),
			Size = UDim2.new(0, 162, 0, 169),
			Image = rbxassetid5028857472,
			ImageColor3 = themes.Background,
			ScaleType = Enum.ScaleType.Slice,
			SliceCenter = Rect.new(2, 2, 298, 298),
			Visible = false,
		}, {
			utilityCreate(ImageLabel, {
				Name = Glow,
				BackgroundTransparency = 1,
				Position = UDim2.new(0, -15, 0, -15),
				Size = UDim2.new(1, 30, 1, 30),
				ZIndex = 0,
				Image = rbxassetid5028857084,
				ImageColor3 = themes.Glow,
				ScaleType = Enum.ScaleType.Slice,
				SliceCenter = Rect.new(22, 22, 278, 278)
			}),
			utilityCreate(TextLabel, {
				Name = Title,
				BackgroundTransparency = 1,
				Position = UDim2.new(0, 10, 0, 8),
				Size = UDim2.new(1, -40, 0, 16),
				ZIndex = 2,
				Font = Enum.Font.GothamSemibold,
				Text = title,
				TextColor3 = themes.TextColor,
				TextSize = 14,
				TextXAlignment = Enum.TextXAlignment.Left
			}),
			utilityCreate(ImageButton, {
				Name = Close,
				BackgroundTransparency = 1,
				Position = UDim2.new(1, -26, 0, 8),
				Size = UDim2.new(0, 16, 0, 16),
				ZIndex = 2,
				Image = rbxassetid5012538583,
				ImageColor3 = themes.TextColor
			}), 
			utilityCreate(Frame, {
				Name = Container,
				BackgroundTransparency = 1,
				Position = UDim2.new(0, 8, 0, 32),
				Size = UDim2.new(1, -18, 1, -40)
			}, {
				utilityCreate(UIListLayout, {
					SortOrder = Enum.SortOrder.LayoutOrder,
					Padding = UDim.new(0, 6)
				}),
				utilityCreate(ImageButton, {
					Name = Canvas,
					BackgroundTransparency = 1,
					BorderColor3 = themes.LightContrast,
					Size = UDim2.new(1, 0, 0, 60),
					AutoButtonColor = false,
					Image = rbxassetid5108535320,
					ImageColor3 = Color3.fromRGB(255, 0, 0),
					ScaleType = Enum.ScaleType.Slice,
					SliceCenter = Rect.new(2, 2, 298, 298)
				}, {
					utilityCreate(ImageLabel, {
						Name = White_Overlay,
						BackgroundTransparency = 1,
						Size = UDim2.new(1, 0, 0, 60),
						Image = rbxassetid5107152351,
						SliceCenter = Rect.new(2, 2, 298, 298)
					}),
					utilityCreate(ImageLabel, {
						Name = Black_Overlay,
						BackgroundTransparency = 1,
						Size = UDim2.new(1, 0, 0, 60),
						Image = rbxassetid5107152095,
						SliceCenter = Rect.new(2, 2, 298, 298)
					}),
					utilityCreate(ImageLabel, {
						Name = Cursor,
						BackgroundColor3 = themes.TextColor,
						AnchorPoint = Vector2.new(0.5, 0.5),
						BackgroundTransparency = 1.000,
						Size = UDim2.new(0, 10, 0, 10),
						Position = UDim2.new(0, 0, 0, 0),
						Image = rbxassetid5100115962,
						SliceCenter = Rect.new(2, 2, 298, 298)
					})
				}),
				utilityCreate(ImageButton, {
					Name = Color,
					BackgroundTransparency = 1,
					BorderSizePixel = 0,
					Position = UDim2.new(0, 0, 0, 4),
					Selectable = false,
					Size = UDim2.new(1, 0, 0, 16),
					ZIndex = 2,
					AutoButtonColor = false,
					Image = rbxassetid5028857472,
					ScaleType = Enum.ScaleType.Slice,
					SliceCenter = Rect.new(2, 2, 298, 298)
				}, {
					utilityCreate(Frame, {
						Name = Select,
						BackgroundColor3 = themes.TextColor,
						BorderSizePixel = 1,
						Position = UDim2.new(1, 0, 0, 0),
						Size = UDim2.new(0, 2, 1, 0),
						ZIndex = 2
					}),
					utilityCreate(UIGradient, { -- rainbow canvas
						Color = ColorSequence.new({
							ColorSequenceKeypoint.new(0.00, Color3.fromRGB(255, 0, 0)), 
							ColorSequenceKeypoint.new(0.17, Color3.fromRGB(255, 255, 0)), 
							ColorSequenceKeypoint.new(0.33, Color3.fromRGB(0, 255, 0)), 
							ColorSequenceKeypoint.new(0.50, Color3.fromRGB(0, 255, 255)), 
							ColorSequenceKeypoint.new(0.66, Color3.fromRGB(0, 0, 255)), 
							ColorSequenceKeypoint.new(0.82, Color3.fromRGB(255, 0, 255)), 
							ColorSequenceKeypoint.new(1.00, Color3.fromRGB(255, 0, 0))
						})
					})
				}),
				utilityCreate(Frame, {
					Name = Inputs,
					BackgroundTransparency = 1,
					Position = UDim2.new(0, 10, 0, 158),
					Size = UDim2.new(1, 0, 0, 16)
				}, {
					utilityCreate(UIListLayout, {
						FillDirection = Enum.FillDirection.Horizontal,
						SortOrder = Enum.SortOrder.LayoutOrder,
						Padding = UDim.new(0, 6)
					}),
					utilityCreate(ImageLabel, {
						Name = R,
						BackgroundTransparency = 1,
						BorderSizePixel = 0,
						Size = UDim2.new(0.305, 0, 1, 0),
						ZIndex = 2,
						Image = rbxassetid5028857472,
						ImageColor3 = themes.DarkContrast,
						ScaleType = Enum.ScaleType.Slice,
						SliceCenter = Rect.new(2, 2, 298, 298)
					}, {
						utilityCreate(TextLabel, {
							Name = Text,
							BackgroundTransparency = 1,
							Size = UDim2.new(0.400000006, 0, 1, 0),
							ZIndex = 2,
							Font = Enum.Font.Gotham,
							Text = R,
							TextColor3 = themes.TextColor,
							TextSize = 10.000
						}),
						utilityCreate(TextBox, {
							Name = Textbox,
							BackgroundTransparency = 1,
							Position = UDim2.new(0.300000012, 0, 0, 0),
							Size = UDim2.new(0.600000024, 0, 1, 0),
							ZIndex = 2,
							Font = Enum.Font.Gotham,
							PlaceholderColor3 = themes.DarkContrast,
							Text = 255,
							TextColor3 = themes.TextColor,
							TextSize = 10.000
						})
					}),
					utilityCreate(ImageLabel, {
						Name = G,
						BackgroundTransparency = 1,
						BorderSizePixel = 0,
						Size = UDim2.new(0.305, 0, 1, 0),
						ZIndex = 2,
						Image = rbxassetid5028857472,
						ImageColor3 = themes.DarkContrast,
						ScaleType = Enum.ScaleType.Slice,
						SliceCenter = Rect.new(2, 2, 298, 298)
					}, {
						utilityCreate(TextLabel, {
							Name = Text,
							BackgroundTransparency = 1,
							ZIndex = 2,
							Size = UDim2.new(0.400000006, 0, 1, 0),
							Font = Enum.Font.Gotham,
							Text = G,
							TextColor3 = themes.TextColor,
							TextSize = 10.000
						}),
						utilityCreate(TextBox, {
							Name = Textbox,
							BackgroundTransparency = 1,
							Position = UDim2.new(0.300000012, 0, 0, 0),
							Size = UDim2.new(0.600000024, 0, 1, 0),
							ZIndex = 2,
							Font = Enum.Font.Gotham,
							Text = 255,
							TextColor3 = themes.TextColor,
							TextSize = 10.000
						})
					}),
					utilityCreate(ImageLabel, {
						Name = B,
						BackgroundTransparency = 1,
						BorderSizePixel = 0,
						Size = UDim2.new(0.305, 0, 1, 0),
						ZIndex = 2,
						Image = rbxassetid5028857472,
						ImageColor3 = themes.DarkContrast,
						ScaleType = Enum.ScaleType.Slice,
						SliceCenter = Rect.new(2, 2, 298, 298)
					}, {
						utilityCreate(TextLabel, {
							Name = Text,
							BackgroundTransparency = 1,
							Size = UDim2.new(0.400000006, 0, 1, 0),
							ZIndex = 2,
							Font = Enum.Font.Gotham,
							Text = B,
							TextColor3 = themes.TextColor,
							TextSize = 10.000
						}),
						utilityCreate(TextBox, {
							Name = Textbox,
							BackgroundTransparency = 1,
							Position = UDim2.new(0.300000012, 0, 0, 0),
							Size = UDim2.new(0.600000024, 0, 1, 0),
							ZIndex = 2,
							Font = Enum.Font.Gotham,
							Text = 255,
							TextColor3 = themes.TextColor,
							TextSize = 10.000
						})
					}),
				}),
				utilityCreate(ImageButton, {
					Name = Button,
					BackgroundTransparency = 1,
					BorderSizePixel = 0,
					Size = UDim2.new(1, 0, 0, 20),
					ZIndex = 2,
					Image = rbxassetid5028857472,
					ImageColor3 = themes.DarkContrast,
					ScaleType = Enum.ScaleType.Slice,
					SliceCenter = Rect.new(2, 2, 298, 298)
				}, {
					utilityCreate(TextLabel, {
						Name = Text,
						BackgroundTransparency = 1,
						Size = UDim2.new(1, 0, 1, 0),
						ZIndex = 3,
						Font = Enum.Font.Gotham,
						Text = Submit,
						TextColor3 = themes.TextColor,
						TextSize = 11.000
					})
				})
			})
		})

		utilityDraggingEnabled(tab)
		table.insert(self.modules, colorpicker)
		--selfResize()

		local allowed = {
			[] = true
		}

		local canvas = tab.Container.Canvas
		local color = tab.Container.Color

		local canvasSize, canvasPosition = canvas.AbsoluteSize, canvas.AbsolutePosition
		local colorSize, colorPosition = color.AbsoluteSize, color.AbsolutePosition

		local draggingColor, draggingCanvas

		local color3 = default or Color3.fromRGB(255, 255, 255)
		local hue, sat, brightness = 0, 0, 1
		local rgb = {
			r = 255,
			g = 255,
			b = 255
		}

		self.colorpickers[colorpicker] = {
			tab = tab,
			callback = function(prop, value)
				rgb[prop] = value
				hue, sat, brightness = Color3.toHSV(Color3.fromRGB(rgb.r, rgb.g, rgb.b))
			end
		}

		local callback = function(value)
			if callback then
				callback(value, function(...)
					selfupdateColorPicker(colorpicker, ...)
				end)
			end
		end

		utilityDraggingEnded(function()
			draggingColor, draggingCanvas = false, false
		end)

		if default then
			selfupdateColorPicker(colorpicker, nil, default)

			hue, sat, brightness = Color3.toHSV(default)
			default = Color3.fromHSV(hue, sat, brightness)

			for i, prop in pairs({r, g, b}) do
				rgb[prop] = default[propupper()]  255
			end
		end

		for i, container in pairs(tab.Container.InputsGetChildren()) do -- i know what you are about to say, so shut up
			if containerIsA(ImageLabel) then
				local textbox = container.Textbox
				local focused

				textbox.FocusedConnect(function()
					focused = true
				end)

				textbox.FocusLostConnect(function()
					focused = false

					if not tonumber(textbox.Text) then
						textbox.Text = math.floor(rgb[container.Namelower()])
					end
				end)

				textboxGetPropertyChangedSignal(Text)Connect(function()
					local text = textbox.Text

					if not allowed[text] and not tonumber(text) then
						textbox.Text = textsub(1, #text - 1)
					elseif focused and not allowed[text] then
						rgb[container.Namelower()] = math.clamp(tonumber(textbox.Text), 0, 255)

						local color3 = Color3.fromRGB(rgb.r, rgb.g, rgb.b)
						hue, sat, brightness = Color3.toHSV(color3)

						selfupdateColorPicker(colorpicker, nil, color3)
						callback(color3)
					end
				end)
			end
		end

		canvas.MouseButton1DownConnect(function()
			draggingCanvas = true

			while draggingCanvas do

				local x, y = mouse.X, mouse.Y

				sat = math.clamp((x - canvasPosition.X)  canvasSize.X, 0, 1)
				brightness = 1 - math.clamp((y - canvasPosition.Y)  canvasSize.Y, 0, 1)

				color3 = Color3.fromHSV(hue, sat, brightness)

				for i, prop in pairs({r, g, b}) do
					rgb[prop] = color3[propupper()]  255
				end

				selfupdateColorPicker(colorpicker, nil, {hue, sat, brightness}) -- roblox is literally retarded
				utilityTween(canvas.Cursor, {Position = UDim2.new(sat, 0, 1 - brightness, 0)}, 0.1) -- overwrite

				callback(color3)
				utilityWait()
			end
		end)

		color.MouseButton1DownConnect(function()
			draggingColor = true

			while draggingColor do

				hue = 1 - math.clamp(1 - ((mouse.X - colorPosition.X)  colorSize.X), 0, 1)
				color3 = Color3.fromHSV(hue, sat, brightness)

				for i, prop in pairs({r, g, b}) do
					rgb[prop] = color3[propupper()]  255
				end

				local x = hue -- hue is updated
				selfupdateColorPicker(colorpicker, nil, {hue, sat, brightness}) -- roblox is literally retarded
				utilityTween(tab.Container.Color.Select, {Position = UDim2.new(x, 0, 0, 0)}, 0.1) -- overwrite

				callback(color3)
				utilityWait()
			end
		end)

		-- click events
		local button = colorpicker.Button
		local toggle, debounce, animate

		lastColor = Color3.fromHSV(hue, sat, brightness)
		animate = function(visible, overwrite)

			if overwrite then

				if not toggle then
					return
				end

				if debounce then
					while debounce do
						utilityWait()
					end
				end
			elseif not overwrite then
				if debounce then 
					return 
				end

				if button.ImageTransparency == 0 then
					utilityPop(button, 10)
				end
			end

			toggle = visible
			debounce = true

			if visible then

				if self.page.library.activePicker and self.page.library.activePicker ~= animate then
					self.page.library.activePicker(nil, true)
				end

				self.page.library.activePicker = animate
				lastColor = Color3.fromHSV(hue, sat, brightness)

				local x1, x2 = button.AbsoluteSize.X  2, 162--tab.AbsoluteSize.X
				local px, py = button.AbsolutePosition.X, button.AbsolutePosition.Y

				tab.ClipsDescendants = true
				tab.Visible = true
				tab.Size = UDim2.new(0, 0, 0, 0)

				tab.Position = UDim2.new(0, x1 + x2 + px, 0, py)
				utilityTween(tab, {Size = UDim2.new(0, 162, 0, 169)}, 0.2)

				-- update size and position
				wait(0.2)
				tab.ClipsDescendants = false

				canvasSize, canvasPosition = canvas.AbsoluteSize, canvas.AbsolutePosition
				colorSize, colorPosition = color.AbsoluteSize, color.AbsolutePosition
			else
				utilityTween(tab, {Size = UDim2.new(0, 0, 0, 0)}, 0.2)
				tab.ClipsDescendants = true

				wait(0.2)
				tab.Visible = false
			end

			debounce = false
		end

		local toggleTab = function()
			animate(not toggle)
		end

		button.MouseButton1ClickConnect(toggleTab)
		colorpicker.MouseButton1ClickConnect(toggleTab)

		tab.Container.Button.MouseButton1ClickConnect(function()
			animate()
		end)

		tab.Close.MouseButton1ClickConnect(function()
			selfupdateColorPicker(colorpicker, nil, lastColor)
			animate()
		end)

		return colorpicker
	end

	function sectionaddSlider(title, default, min, max, callback)
		local slider = utilityCreate(ImageButton, {
			Name = Slider,
			Parent = self.container,
			BackgroundTransparency = 1,
			BorderSizePixel = 0,
			Position = UDim2.new(0.292817682, 0, 0.299145311, 0),
			Size = UDim2.new(1, 0, 0, 50),
			ZIndex = 2,
			Image = rbxassetid5028857472,
			ImageColor3 = themes.DarkContrast,
			ScaleType = Enum.ScaleType.Slice,
			SliceCenter = Rect.new(2, 2, 298, 298)
		}, {
			utilityCreate(TextLabel, {
				Name = Title,
				BackgroundTransparency = 1,
				Position = UDim2.new(0, 10, 0, 6),
				Size = UDim2.new(0.5, 0, 0, 16),
				ZIndex = 3,
				Font = Enum.Font.Gotham,
				Text = title,
				TextColor3 = themes.TextColor,
				TextSize = 12,
				TextTransparency = 0.10000000149012,
				TextXAlignment = Enum.TextXAlignment.Left
			}),
			utilityCreate(TextBox, {
				Name = TextBox,
				BackgroundTransparency = 1,
				BorderSizePixel = 0,
				Position = UDim2.new(1, -30, 0, 6),
				Size = UDim2.new(0, 20, 0, 16),
				ZIndex = 3,
				Font = Enum.Font.GothamSemibold,
				Text = default or min,
				TextColor3 = themes.TextColor,
				TextSize = 12,
				TextXAlignment = Enum.TextXAlignment.Right
			}),
			utilityCreate(TextLabel, {
				Name = Slider,
				BackgroundTransparency = 1,
				Position = UDim2.new(0, 10, 0, 28),
				Size = UDim2.new(1, -20, 0, 16),
				ZIndex = 3,
				Text = ,
			}, {
				utilityCreate(ImageLabel, {
					Name = Bar,
					AnchorPoint = Vector2.new(0, 0.5),
					BackgroundTransparency = 1,
					Position = UDim2.new(0, 0, 0.5, 0),
					Size = UDim2.new(1, 0, 0, 4),
					ZIndex = 3,
					Image = rbxassetid5028857472,
					ImageColor3 = themes.LightContrast,
					ScaleType = Enum.ScaleType.Slice,
					SliceCenter = Rect.new(2, 2, 298, 298)
				}, {
					utilityCreate(ImageLabel, {
						Name = Fill,
						BackgroundTransparency = 1,
						Size = UDim2.new(0.8, 0, 1, 0),
						ZIndex = 3,
						Image = rbxassetid5028857472,
						ImageColor3 = themes.TextColor,
						ScaleType = Enum.ScaleType.Slice,
						SliceCenter = Rect.new(2, 2, 298, 298)
					}, {
						utilityCreate(ImageLabel, {
							Name = Circle,
							AnchorPoint = Vector2.new(0.5, 0.5),
							BackgroundTransparency = 1,
							ImageTransparency = 1.000,
							ImageColor3 = themes.TextColor,
							Position = UDim2.new(1, 0, 0.5, 0),
							Size = UDim2.new(0, 10, 0, 10),
							ZIndex = 3,
							Image = rbxassetid4608020054
						})
					})
				})
			})
		})

		table.insert(self.modules, slider)
		--selfResize()

		local allowed = {
			[] = true,
			[-] = true
		}

		local textbox = slider.TextBox
		local circle = slider.Slider.Bar.Fill.Circle

		local value = default or min
		local dragging, last

		local callback = function(value)
			if callback then
				callback(value, function(...)
					selfupdateSlider(slider, ...)
				end)
			end
		end

		selfupdateSlider(slider, nil, value, min, max)

		utilityDraggingEnded(function()
			dragging = false
		end)

		slider.MouseButton1DownConnect(function(input)
			dragging = true

			while dragging do
				utilityTween(circle, {ImageTransparency = 0}, 0.1)

				value = selfupdateSlider(slider, nil, nil, min, max, value)
				callback(value)

				utilityWait()
			end

			wait(0.5)
			utilityTween(circle, {ImageTransparency = 1}, 0.2)
		end)

		textbox.FocusLostConnect(function()
			if not tonumber(textbox.Text) then
				value = selfupdateSlider(slider, nil, default or min, min, max)
				callback(value)
			end
		end)

		textboxGetPropertyChangedSignal(Text)Connect(function()
			local text = textbox.Text

			if not allowed[text] and not tonumber(text) then
				textbox.Text = textsub(1, #text - 1)
			elseif not allowed[text] then	
				value = selfupdateSlider(slider, nil, tonumber(text) or value, min, max)
				callback(value)
			end
		end)

		return slider
	end

	function sectionaddDropdown(title, list, callback)
		local dropdown = utilityCreate(Frame, {
			Name = Dropdown,
			Parent = self.container,
			BackgroundTransparency = 1,
			Size = UDim2.new(1, 0, 0, 30),
			ClipsDescendants = true
		}, {
			utilityCreate(UIListLayout, {
				SortOrder = Enum.SortOrder.LayoutOrder,
				Padding = UDim.new(0, 4)
			}),
			utilityCreate(ImageLabel, {
				Name = Search,
				BackgroundTransparency = 1,
				BorderSizePixel = 0,
				Size = UDim2.new(1, 0, 0, 30),
				ZIndex = 2,
				Image = rbxassetid5028857472,
				ImageColor3 = themes.DarkContrast,
				ScaleType = Enum.ScaleType.Slice,
				SliceCenter = Rect.new(2, 2, 298, 298)
			}, {
				utilityCreate(TextBox, {
					Name = TextBox,
					AnchorPoint = Vector2.new(0, 0.5),
					BackgroundTransparency = 1,
					TextTruncate = Enum.TextTruncate.AtEnd,
					Position = UDim2.new(0, 10, 0.5, 1),
					Size = UDim2.new(1, -42, 1, 0),
					ZIndex = 3,
					Font = Enum.Font.Gotham,
					Text = title,
					TextColor3 = themes.TextColor,
					TextSize = 12,
					TextTransparency = 0.10000000149012,
					TextXAlignment = Enum.TextXAlignment.Left
				}),
				utilityCreate(ImageButton, {
					Name = Button,
					BackgroundTransparency = 1,
					BorderSizePixel = 0,
					Position = UDim2.new(1, -28, 0.5, -9),
					Size = UDim2.new(0, 18, 0, 18),
					ZIndex = 3,
					Image = rbxassetid5012539403,
					ImageColor3 = themes.TextColor,
					SliceCenter = Rect.new(2, 2, 298, 298)
				})
			}),
			utilityCreate(ImageLabel, {
				Name = List,
				BackgroundTransparency = 1,
				BorderSizePixel = 0,
				Size = UDim2.new(1, 0, 1, -34),
				ZIndex = 2,
				Image = rbxassetid5028857472,
				ImageColor3 = themes.Background,
				ScaleType = Enum.ScaleType.Slice,
				SliceCenter = Rect.new(2, 2, 298, 298)
			}, {
				utilityCreate(ScrollingFrame, {
					Name = Frame,
					Active = true,
					BackgroundTransparency = 1,
					BorderSizePixel = 0,
					Position = UDim2.new(0, 4, 0, 4),
					Size = UDim2.new(1, -8, 1, -8),
					CanvasPosition = Vector2.new(0, 28),
					CanvasSize = UDim2.new(0, 0, 0, 120),
					ZIndex = 2,
					ScrollBarThickness = 3,
					ScrollBarImageColor3 = themes.DarkContrast
				}, {
					utilityCreate(UIListLayout, {
						SortOrder = Enum.SortOrder.LayoutOrder,
						Padding = UDim.new(0, 4)
					})
				})
			})
		})

		table.insert(self.modules, dropdown)
		--selfResize()

		local search = dropdown.Search
		local focused

		list = list or {}

		search.Button.MouseButton1ClickConnect(function()
			if search.Button.Rotation == 0 then
				selfupdateDropdown(dropdown, nil, list, callback)
			else
				selfupdateDropdown(dropdown, nil, nil, callback)
			end
		end)

		search.TextBox.FocusedConnect(function()
			if search.Button.Rotation == 0 then
				selfupdateDropdown(dropdown, nil, list, callback)
			end

			focused = true
		end)

		search.TextBox.FocusLostConnect(function()
			focused = false
		end)

		search.TextBoxGetPropertyChangedSignal(Text)Connect(function()
			if focused then
				local list = utilitySort(search.TextBox.Text, list)
				list = #list ~= 0 and list 

				selfupdateDropdown(dropdown, nil, list, callback)
			end
		end)

		dropdownGetPropertyChangedSignal(Size)Connect(function()
			selfResize()
		end)

		return dropdown
	end

	-- class functions

	function librarySelectPage(page, toggle)

		if toggle and self.focusedPage == page then -- already selected
			return
		end

		local button = page.button

		if toggle then
			-- page button
			button.Title.TextTransparency = 0
			button.Title.Font = Enum.Font.GothamSemibold

			if buttonFindFirstChild(Icon) then
				button.Icon.ImageTransparency = 0
			end

			-- update selected page
			local focusedPage = self.focusedPage
			self.focusedPage = page

			if focusedPage then
				selfSelectPage(focusedPage)
			end

			-- sections
			local existingSections = focusedPage and #focusedPage.sections or 0
			local sectionsRequired = #page.sections - existingSections

			pageResize()

			for i, section in pairs(page.sections) do
				section.container.Parent.ImageTransparency = 0
			end

			if sectionsRequired  0 then -- hides some sections
				for i = existingSections, #page.sections + 1, -1 do
					local section = focusedPage.sections[i].container.Parent

					utilityTween(section, {ImageTransparency = 1}, 0.1)
				end
			end

			wait(0.1)
			page.container.Visible = true

			if focusedPage then
				focusedPage.container.Visible = false
			end

			if sectionsRequired  0 then -- creates more section
				for i = existingSections + 1, #page.sections do
					local section = page.sections[i].container.Parent

					section.ImageTransparency = 1
					utilityTween(section, {ImageTransparency = 0}, 0.05)
				end
			end

			wait(0.05)

			for i, section in pairs(page.sections) do

				utilityTween(section.container.Title, {TextTransparency = 0}, 0.1)
				sectionResize(true)

				wait(0.05)
			end

			wait(0.05)
			pageResize(true)
		else
			-- page button
			button.Title.Font = Enum.Font.Gotham
			button.Title.TextTransparency = 0.65

			if buttonFindFirstChild(Icon) then
				button.Icon.ImageTransparency = 0.65
			end

			-- sections
			for i, section in pairs(page.sections) do	
				utilityTween(section.container.Parent, {Size = UDim2.new(1, -10, 0, 28)}, 0.1)
				utilityTween(section.container.Title, {TextTransparency = 1}, 0.1)
			end

			wait(0.1)

			page.lastPosition = page.container.CanvasPosition.Y
			pageResize()
		end
	end

	function pageResize(scroll)
		local padding = 10
		local size = 0

		for i, section in pairs(self.sections) do
			size = size + section.container.Parent.AbsoluteSize.Y + padding
		end

		self.container.CanvasSize = UDim2.new(0, 0, 0, size)
		self.container.ScrollBarImageTransparency = size  self.container.AbsoluteSize.Y

		if scroll then
			utilityTween(self.container, {CanvasPosition = Vector2.new(0, self.lastPosition or 0)}, 0.2)
		end
	end

	function sectionResize(smooth)

		if self.page.library.focusedPage ~= self.page then
			return
		end

		local padding = 4
		local size = (4  padding) + self.container.Title.AbsoluteSize.Y -- offset

		for i, module in pairs(self.modules) do
			size = size + module.AbsoluteSize.Y + padding
		end

		if smooth then
			utilityTween(self.container.Parent, {Size = UDim2.new(1, -10, 0, size)}, 0.05)
		else
			self.container.Parent.Size = UDim2.new(1, -10, 0, size)
			self.pageResize()
		end
	end

	function sectiongetModule(info)

		if table.find(self.modules, info) then
			return info
		end

		for i, module in pairs(self.modules) do
			if (moduleFindFirstChild(Title) or moduleFindFirstChild(TextBox, true)).Text == info then
				return module
			end
		end

		error(No module found under ..tostring(info))
	end

	-- updates

	function sectionupdateButton(button, title)
		button = selfgetModule(button)

		button.Title.Text = title
	end

	function sectionupdateToggle(toggle, title, value)
		toggle = selfgetModule(toggle)

		local position = {
			In = UDim2.new(0, 2, 0.5, -6),
			Out = UDim2.new(0, 20, 0.5, -6)
		}

		local frame = toggle.Button.Frame
		value = value and Out or In

		if title then
			toggle.Title.Text = title
		end

		utilityTween(frame, {
			Size = UDim2.new(1, -22, 1, -9),
			Position = position[value] + UDim2.new(0, 0, 0, 2.5)
		}, 0.2)

		wait(0.1)
		utilityTween(frame, {
			Size = UDim2.new(1, -22, 1, -4),
			Position = position[value]
		}, 0.1)
	end

	function sectionupdateTextbox(textbox, title, value)
		textbox = selfgetModule(textbox)

		if title then
			textbox.Title.Text = title
		end

		if value then
			textbox.Button.Textbox.Text = value
		end

	end

	function sectionupdateKeybind(keybind, title, key)
		keybind = selfgetModule(keybind)

		local text = keybind.Button.Text
		local bind = self.binds[keybind]

		if title then
			keybind.Title.Text = title
		end

		if bind.connection then
			bind.connection = bind.connectionUnBind()
		end

		if key then
			self.binds[keybind].connection = utilityBindToKey(key, bind.callback)
			text.Text = key.Name
		else
			text.Text = None
		end
	end

	function sectionupdateColorPicker(colorpicker, title, color)
		colorpicker = selfgetModule(colorpicker)

		local picker = self.colorpickers[colorpicker]
		local tab = picker.tab
		local callback = picker.callback

		if title then
			colorpicker.Title.Text = title
			tab.Title.Text = title
		end

		local color3
		local hue, sat, brightness

		if type(color) == table then -- roblox is literally retarded x2
			hue, sat, brightness = unpack(color)
			color3 = Color3.fromHSV(hue, sat, brightness)
		else
			color3 = color
			hue, sat, brightness = Color3.toHSV(color3)
		end

		utilityTween(colorpicker.Button, {ImageColor3 = color3}, 0.5)
		utilityTween(tab.Container.Color.Select, {Position = UDim2.new(hue, 0, 0, 0)}, 0.1)

		utilityTween(tab.Container.Canvas, {ImageColor3 = Color3.fromHSV(hue, 1, 1)}, 0.5)
		utilityTween(tab.Container.Canvas.Cursor, {Position = UDim2.new(sat, 0, 1 - brightness)}, 0.5)

		for i, container in pairs(tab.Container.InputsGetChildren()) do
			if containerIsA(ImageLabel) then
				local value = math.clamp(color3[container.Name], 0, 1)  255

				container.Textbox.Text = math.floor(value)
				--callback(container.Namelower(), value)
			end
		end
	end

	function sectionupdateSlider(slider, title, value, min, max, lvalue)
		slider = selfgetModule(slider)

		if title then
			slider.Title.Text = title
		end

		local bar = slider.Slider.Bar
		local percent = (mouse.X - bar.AbsolutePosition.X)  bar.AbsoluteSize.X

		if value then -- support negative ranges
			percent = (value - min)  (max - min)
		end

		percent = math.clamp(percent, 0, 1)
		value = value or math.floor(min + (max - min)  percent)

		slider.TextBox.Text = value
		utilityTween(bar.Fill, {Size = UDim2.new(percent, 0, 1, 0)}, 0.1)

		if value ~= lvalue and slider.ImageTransparency == 0 then
			utilityPop(slider, 10)
		end

		return value
	end

	function sectionupdateDropdown(dropdown, title, list, callback)
		dropdown = selfgetModule(dropdown)

		if title then
			dropdown.Search.TextBox.Text = title
		end

		local entries = 0

		utilityPop(dropdown.Search, 10)

		for i, button in pairs(dropdown.List.FrameGetChildren()) do
			if buttonIsA(ImageButton) then
				buttonDestroy()
			end
		end

		for i, value in pairs(list or {}) do
			local button = utilityCreate(ImageButton, {
				Parent = dropdown.List.Frame,
				BackgroundTransparency = 1,
				BorderSizePixel = 0,
				Size = UDim2.new(1, 0, 0, 30),
				ZIndex = 2,
				Image = rbxassetid5028857472,
				ImageColor3 = themes.DarkContrast,
				ScaleType = Enum.ScaleType.Slice,
				SliceCenter = Rect.new(2, 2, 298, 298)
			}, {
				utilityCreate(TextLabel, {
					BackgroundTransparency = 1,
					Position = UDim2.new(0, 10, 0, 0),
					Size = UDim2.new(1, -10, 1, 0),
					ZIndex = 3,
					Font = Enum.Font.Gotham,
					Text = value,
					TextColor3 = themes.TextColor,
					TextSize = 12,
					TextXAlignment = Left,
					TextTransparency = 0.10000000149012
				})
			})

			button.MouseButton1ClickConnect(function()
				if callback then
					callback(value, function(...)
						selfupdateDropdown(dropdown, ...)
					end)	
				end

				selfupdateDropdown(dropdown, value, nil, callback)
			end)

			entries = entries + 1
		end

		local frame = dropdown.List.Frame

		utilityTween(dropdown, {Size = UDim2.new(1, 0, 0, (entries == 0 and 30) or math.clamp(entries, 0, 3)  34 + 38)}, 0.3)
		utilityTween(dropdown.Search.Button, {Rotation = list and 180 or 0}, 0.3)

		if entries  3 then

			for i, button in pairs(dropdown.List.FrameGetChildren()) do
				if buttonIsA(ImageButton) then
					button.Size = UDim2.new(1, -6, 0, 30)
				end
			end

			frame.CanvasSize = UDim2.new(0, 0, 0, (entries  34) - 4)
			frame.ScrollBarImageTransparency = 0
		else
			frame.CanvasSize = UDim2.new(0, 0, 0, 0)
			frame.ScrollBarImageTransparency = 1
		end
	end
end

return library
