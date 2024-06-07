 # vue-stay-canvas

 stay-canvas for vue

 <div align="center"><a href="./README.en.md">
   <strong>English</strong>
 </a> | <a href="./README.zh.md">
   <strong>中文简体</strong>
 </a></div>

 ## Translated by [ChatGPT4](https://chatgpt.com/)


 ## Table of Contents

 - [Introduction](#introduction)
 - [Main Features](#main-features)
 - [Installation](#installation)
 - [Getting Started Example](#getting-started-example)
 - [More Examples](#more-examples)
 - [Core Concepts](#core-concepts)

   - [Shape](#shape)
   - [Listener](#listener)
   - [Event](#event)

 - [API Documentation](#api-documentation)

   - [StayCanvas Component](#staycanvas-component-api)
   - [Shape API](#shape-api)

     - [Image](#image)
     - [Point](#point)
     - [Line](#line)
     - [Rectangle](#rectangle)
     - [Circle](#circle)
     - [Text](#text)
     - [Path](#path)
     - [Custom Shape](#custom-shape)
     - [Shape State](#shape-state)
     - [Animation](#animation)

   - [Listener API](#listener-api)

     - [Selector](#selector)
     - [State](#state)
     - [Simple Logic Operations](#simple-logic-operations)
     - [Event](#event)
     - [Listener Callback Function](#listener-callback-function)
     - [StayTools Utility Functions](#staytools-utility-functions)

       - [createChild](#createchild)
       - [updateChild](#updatechild)
       - [removeChild](#removechild)
       - [getContainPointChildren](#getcontainpointchildren)
       - [hasChild](#haschild)
       - [fix](#fix)
       - [switchState](#switchstate)
       - [getChildrenBySelector](#getchildrenbyselector)
       - [getAvailableStates](#getavailablestates)
       - [changeCursor](#changecursor)
       - [moveStart](#movestart)
       - [move](#move)
       - [zoom](#zoom)
       - [log](#log)
       - [redo](#redo)
       - [undo](#undo)
       - [triggerAction](#triggeraction)
       - [deleteListener](#deletelistener)

   - [Event API](#event-api)

     - [name](#name)
     - [trigger](#trigger)
     - [conditionCallback](#conditioncallback)
     - [successCallback](#successcallback)

   - [trigger Function API](#trigger-function-api)

 ## Introduction

 `vue-stay-canvas` provides a set of easy-to-use APIs to help developers integrate canvas functionality into Vue projects. Whether it's drag-and-drop operations, shape drawing, or complex event handling, this component can meet your needs.

 ## Main Features

 - **Quick Start**: Developers can quickly get started and easily implement various graphics and interaction effects.
 - **Flexible and Powerful Configurability**: Supports custom events, custom listeners, and custom drawing components, allowing developers to highly customize according to specific needs.
 - **Rich Graphics Support**: Supports various basic shapes such as rectangles, circles, paths, images, etc.
 - **Easy Integration**: Simple API design makes it quickly integrated into existing Vue projects.
 - **Zero Dependencies**: No third-party dependencies, but Vue installation is required.

 `vue-stay-canvas` allows you to easily achieve various graphics and interaction effects in Vue without deep understanding of the complex Canvas API.

 ## Installation

 ```bash
 npm install vue-stay-canvas
 ```

 ## Getting Started Example

 ```typescript
 <script setup lang="ts">
 import { Rectangle, StayCanvas, type ListenerProps } from 'vue-stay-canvas'
 const DragListener: ListenerProps = {
   name: 'dragListener',
   event: ['dragstart', 'drag', 'dragend'],
   callback: ({ e, composeStore, tools: { appendChild, updateChild } }) => {
     return {
       dragstart: () => ({
         dragStartPosition: e.point,
         dragChild: appendChild({
           shape: new Rectangle({
             x: e.x,
             y: e.y,
             width: 0,
             height: 0,
             props: { color: 'red' }
           }),
           className: 'annotation'
         })
       }),
       drag: () => {
         const { dragStartPosition, dragChild } = composeStore
         const x = Math.min(dragStartPosition.x, e.x)
         const y = Math.min(dragStartPosition.y, e.y)
         const width = Math.abs(dragStartPosition.x - e.x)
         const height = Math.abs(dragStartPosition.y - e.y)
         updateChild({
           child: dragChild,
           shape: dragChild.shape.update({ x, y, width, height })
         })
       }
     }
   }
 }
 </script>
 <template>
   <StayCanvas :width="500" :height="500" :listenerList="[DragListener]" />
 </template>
 ```

 <video src="videos/demo.mp4" controls="">
 </video>

  ## More Examples
  https://github.com/lezhu1234/demo-vue-stay-canvas

 ## Core Concepts

 ### Shape

 In vue-stay-canvas, all elements on the canvas are a `StayChild` object. When using the `createChild`, `appendChild`, `updateChild` functions, this object is returned. Shape is a very important property when creating or updating a `StayChild` object. It accepts a Shape subclass object that defines all drawing behaviors of the object on the canvas. Currently, there are several built-in Shapes in vue-stay-canvas. You can also easily create custom Shapes by inheriting the Shape class.

 - Shape: Base class `StayChild` objects' `Shape` should inherit from this class, and its constructor is defined as follows:

   ```typescript
   constructor({ color, lineWidth, type, gco, state = "default", stateDrawFuncMap = {} }: ShapeProps)

   export interface ShapeProps {
     color?: string | CanvasGradient // The color of the drawing object, this property will be passed to strokeStyle/fillStyle
     lineWidth?: number // The line width of the drawing object, this property will be passed to lineWidth
     type?: valueof<typeof SHAPE_DRAW_TYPES> // "fill" | "stroke", the drawing type of the drawing object
     gco?: GlobalCompositeOperation // The global composite operation of the drawing object, this property will be passed to globalCompositeOperation
     state?: string // The state of the drawing object, used with stateDrawFuncMap to achieve different drawing effects in different states
     stateDrawFuncMap?: Dict<(props: ShapeDrawProps) => void> // The state drawing function set of the drawing object
   }
   ```

 ### Listener

 In vue-stay-canvas, you can register listeners through the listenerList property. This property is an array, and each element in the array is a listener, which is an object that needs to meet the ListenerProps type constraint. Please refer to [Listener API](#listener-api) for details.

 ### Event

 In vue-stay-canvas, you can register events through the eventList. This event list is an array, and each element in the array is an event object that needs to meet the EventProps type constraint. Please refer to [Event API](#event-api) for details.

 ## API Documentation

 ### StayCanvas Component API

 ```typescript
 export interface ContextLayerSetFunction {
   (layer: HTMLCanvasElement): CanvasRenderingContext2D | null
 }

 // The StayCanvas component accepts a StayCanvasProps type props
 export interface StayCanvasProps {
   className?: string // The class of the container, set class style
   width?: number // The width of the container, default is 500px. You must set the width and height of the container here, not in the style
   height?: number // The height of the container, default is 500px
   layers?: number | ContextLayerSetFunction[]  // The number of layers of the container. Each layer will generate a canvas container. The default is 2. You can also pass an array of ContextLayerSetFunction[]. Each element in the array is a function that receives an HTMLCanvasElement type parameter, representing the canvas element corresponding to the layer. You need to return the context2d object corresponding to the layer.
   eventList?: EventProps[] // The event list of the container, which is an array. Each element in the array is an event object. Some events are predefined in vue-stay-canvas. You can override the default events by creating a new event with the same name.
   listenerList?: ListenerProps[] // The listener list of the container, which is an array. Each element in the array is a listener object.
   mounted?: (tools: StayTools) => void // The mount function of the container, which will be executed after the container is mounted.
 }

 // example
 <StayCanvas
   :mounted="init"
   :width="width"
   :height="height"
   :listenerList="listeners"
   :layers="4"
/>

 <StayCanvas
   :mounted="init"
   :width="width"
   :height="height"
   :listenerList="listeners"
   :layers="[
     (canvas) => canvas.getContext("2d"),
     (canvas) =>
       canvas.getContext("2d", {
         willReadFrequently: true,
       }),
   ]"
/>
 ```

 ### Shape API

 #### vue-stay-canvas has built-in simple Shapes, you can easily create custom Shapes by inheriting the Shape class

 - Image

   - This object draws an image on the canvas. Its constructor is defined as follows:

   ```typescript
   // x, y, width, height correspond to dx, dy, dWidth, dHeight in the documentation
   // https://developer.mozilla.org/en-US/docs/Web/API/CanvasRenderingContext2D/drawImage
   export interface ImageProps {
     src: string // The src of the image
     x: number // The x-coordinate of the top-left corner of the image on the canvas, relative to screen pixels
     y: number // The y-coordinate of the top-left corner of the image on the canvas, relative to screen pixels
     width: number // The width of the image when drawn, relative to screen pixels
     height: number // The height of the image when drawn, relative to screen pixels
     sx?: number // The x-coordinate of the start point on the image when drawn, relative to the original pixels of the image
     sy?: number // The y-coordinate of the start point on the image when drawn, relative to the original pixels of the image
     swidth?: number // The width of the image when drawn, relative to the original pixels of the image
     sheight?: number // The height of the image when drawn, relative to the original pixels of the image
     imageLoaded?: (image: StayImage) => void // The callback after the image is loaded
     props?: ShapeProps // This object inherits from Shape
   }
   constructor(imageProps: ImageProps)
   ```

   - The following methods of this object might be useful in some cases:

   ```typescript
   // This method is used to update the properties of the point
   declare update({
     src,
     x,
     y,
     width,
     sx,
     sy,
     swidth,
     sheight,
     height,
     props,
   }: Partial<ImageProps>): this
   ```

 - Point

   - This object draws a point on the canvas. Its constructor is defined as follows:

   ```typescript
   // x:number The x-coordinate of the point
   // y:number The y-coordinate of the point
   // props will be passed to the constructor of Shape
   constructor(x: number, y: number, props: ShapeProps = {})
   ```

   - The following methods of this object might be useful in some cases:

   ```typescript
   // This method is used to update the properties of the point
   declare update({ x, y, props }: PointProps) this

   // This method can calculate the distance between two points
   declare distance(point: Point): number

   // This method can determine whether two points are within a certain distance, which is actually calling the distance method and comparing with offset
   declare near(point: Point, offset: number = 10): boolean

   // This method can determine whether the minimum distance between a point and a line segment is within the specified distance
   // When the projection of the line connecting the point and the endpoint of the line segment on the line segment is on the line segment, the minimum distance is the vertical distance from the point to the line. Otherwise, it is the smaller distance between the point and the two endpoints of the line segment
   // https://en.wikipedia.org/wiki/Distance_from_a_point_to_a_line
   declare nearLine(line: Line, offset: number = 10): boolean
   ```

 - Line

   - This object draws a line segment on the canvas. Its constructor is defined as follows:

   ```typescript
   // x1:number The x-coordinate of the starting point of the line segment
   // y1:number The y-coordinate of the starting point of the line segment
   // x2:number The x-coordinate of the end point of the line segment
   // x2:number The y-coordinate of the end point of the line segment
   // props will be passed to the constructor of Shape
   constructor({ x1, y1, x2, y2, props }: LineProps)
   ```

   - The following methods of this object might be useful in some cases:

   ```typescript

   // This method is used to update the properties of the line segment
   declare update({ x1, y1, x2, y2, props }: UpdateLineProps)this

   // This method is used to calculate the vertical distance from the point to the line
   declare distanceToPoint(point: Point): number

   // This method is used to calculate the length of the line segment
   declare len(): number

   // The calculation method of the nearLine of the Point object is the same
   declare segmentDistanceToPoint(point: Point): number

   // This method can determine whether the minimum distance between a point and a line segment is within the specified distance by calling the segmentDistanceToPoint method
   declare nearPoint(point: Point, offset: number = 10): boolean

   ```

 - Rectangle

   - This object draws a rectangle on the canvas. Its constructor is defined as follows:

   ```typescript
   // x:number The x-coordinate of the top-left corner of the rectangle
   // y:number The y-coordinate of the top-left corner of the rectangle
   // width:number The width of the rectangle
   // height:number The height of the rectangle
   // props will be passed to the constructor of Shape
   constructor({ x, y, width, height, props = {} }: RectangleAttr)
   ```

   - After creation, the following properties will be added to this object:

   ```typescript
   // leftTop: Point The coordinates of the top-left corner of the rectangle
   // rightTop: Point The coordinates of the top-right corner of the rectangle
   // leftBottom: Point The coordinates of the bottom-left corner of the rectangle
   // rightBottom: Point The coordinates of the bottom-right corner of the rectangle
   // leftBorder: Line The line on the left side of the rectangle
   // rightBorder: Line The line on the right side of the rectangle
   // topBorder: Line The line on the top side of the rectangle
   // bottomBorder: Line The line on the bottom side of the rectangle
   // area: number The area of the rectangle
   ```

 - The following methods of this object might be useful in some cases:

   ```typescript
   // This method is used to update the properties of the object
   declare update(Partial<RectangleAttr>): this

   // This method is used to conveniently calculate the scaling ratio and offset required to proportionally scale another rectangle and center it in the current rectangle
   // When calling this method, you need to pass in the width and height values, which will return a new Rectangle object and three attributes
   type FitInfoAttr = {
     rectangle: Rectangle
     scaleRatio: number
     offsetX:number
     offsetY:number
   }
   declare computeFitInfo(width: number, height: number): FitInfoAttr

   // example:
   // Create a rectangle with a width and height of 200*300, and then calculate the scaling ratio and offset required to proportionally scale and center this rectangle in a container rectangle with a width and height of 600*600.
   // rectangle is the new rectangle created by proportional scaling and centering, scaleRatio is the scaling ratio, offsetX and offsetY are the offsets
   const containerRect = new Rectangle({ x: 0, y: 0, width:600, height:600 })
   const { rectangle, scaleRatio, offsetX, offsetY } = containerRect.computeFitInfo(200, 300)

   // This method is used to determine whether a point is within the rectangle
   declare (point: Point): boolean

   // This method is used to copy a rectangle. The copied rectangle will have the same properties as the current rectangle but will not share the same object
   declare copy(): Rectangle

   // These two methods are used to convert the coordinates of the rectangle from the world coordinate system to the screen coordinate system and vice versa
   declare worldToScreen(offsetX: number, offsetY: number, scaleRatio: number): Rectangle
   declare screenToWorld(offsetX: number, offsetY: number, scaleRatio: number):{ x: number, y: number, width: number, height: number }
   ```

 - Circle

   - This object draws a circle on the canvas. Its constructor is defined as follows:

   ```typescript
   export interface CircleAttr {
     x: number // The x-coordinate of the center of the circle
     y: number // The y-coordinate of the center of the circle
     radius: number // The radius of the circle
     props?: ShapeProps
   }
   declare constructor({ x, y, radius, props }: CircleAttr)
   ```

   - The following methods of this object might be useful in some cases:

   ```typescript
   // This method is used to update the properties of the circle
   declare update({ x, y, radius, props }: Partial<CircleAttr>): this

   // This method can determine whether a point is within the circle
   declare (point: Point): boolean
   ```

 - Text

   - This object draws text on the canvas. Its constructor is defined as follows:

   ```typescript
   // x:number The x-coordinate of the text, which is the center x-coordinate of the rectangle containing the text
   // y:number The y-coordinate of the text, which is the center y-coordinate of the rectangle containing the text
   // font:string The font of the text https://developer.mozilla.org/en-US/docs/Web/API/CanvasRenderingContext2D/font
   constructor({ x, y, text, font, props }: TextAttr)
   ```

   - The following methods of this object might be useful in some cases:

   ```typescript
   // This method is used to update the properties of the text
   declare update({
     x,
     y,
     font,
     text,
     props,
   }: Partial<TextAttr>)

   // This method can determine whether a point is within the rectangle where the Text is located
   declare (point: Point): boolean
   ```

 - Path

 - This object draws a path on the canvas. Its constructor is defined as follows:

   ```typescript
   // points: Point[] The points on the path
   // radius: number The radius of the path
   constructor({ points, radius, props }: PathAttr)
   ```

 #### Custom Shape

 - Custom Shape

   - We will take Rectangle as an example to explain how to customize a Shape in detail (Note that this is not the complete Rectangle code, just extracts the necessary parts to introduce the process of customizing Shape)

   ```typescript
   // Your defined class must inherit from Shape and implement the following abstract methods

   // abstract contains(point: SimplePoint, cxt?: CanvasRenderingContext2D): boolean
   // abstract copy(): Shape
   // abstract draw(ctx: CanvasRenderingContext2D, canvas?: HTMLCanvasElement): void
   // abstract move(offsetX: number, offsetY: number): void
   // abstract update(props: any): Shape
   // abstract zoom(zoomScale: number): void
   export class Rectangle extends Shape {
     constructor({ x, y, width, height, props = {} }: RectangleAttr) {
       // You can pass props to Shape here. The properties in props are introduced in the previous module,
       super(props)
       this.x = x
       this.y = y
       this.width = width
       this.height = height
       ...
     }

     ...

     // This function determines whether a point is within your custom Shape. When we define a listener, this function will be called to determine whether the object meets the area condition triggered by the current selector. The parameter of this function is the coordinate of the mouse position
     contains(point: Point): boolean {
       return (
         point.x > this.x &&
         point.x < this.x + this.width &&
         point.y > this.y &&
         point.y < this.y + this.height
       )
     }

     // This function copies the current object. The result needs to be a new object. You can call the _copy() method of Shape to copy props
     copy(): Rectangle {
       return new Rectangle({
         ...this,
         props: this._copy(),
       })
     }

     // The core drawing function. This function draws your Shape. The parameter of this function is the ShapeDrawProps object. You can call the methods of context to draw your Shape

     // export interface ShapeDrawProps {
     //   context: CanvasRenderingContext2D
     //   canvas: HTMLCanvasElement
     //   now: number // Current timestamp, this value can be used to achieve animation effects
     // }
     draw({ context }: ShapeDrawProps) {
       context.lineWidth = this.lineWidth

       if (this.type === SHAPE_DRAW_TYPES.STROKE) {
         context.strokeStyle = this.color
         context.strokeRect(this.x, this.y, this.width, this.height)
       } else if (this.type === SHAPE_DRAW_TYPES.FILL) {
         context.fillStyle = this.color
         context.fillRect(this.x, this.y, this.width, this.height)
       }
     }

     // This function will be called when you need to move the elements on the canvas. If your application does not need this function, you can leave this method empty.
     // This function will be called when moving the entire canvas. You can update the position of your Shape here. The parameter of this function is the offset of the move
     move(offsetX: number, offsetY: number) {
       this.update({
         x: this.x + offsetX,
         y: this.y + offsetY,
       })
     }

     // Update function, used to update the properties of Shape
     update({
       x = this.x,
       y = this.y,
       width = this.width,
       height = this.height,
       props,
     }: Partial<RectangleAttr>) {
       this.x = x
       this.y = y
       this.width = width
       this.height = height
       this._update(props || {})
       this.init()

       return this
     }

     // This function will be called when you need to zoom the elements on the canvas. If your application does not need this function, you can leave this method empty.
     // This function will be called when zooming the entire canvas. You can update the position of your Shape here. The parameter of this function is the zoom ratio. You can use the getZoomPoint method of Shape to calculate the position after zooming for a certain coordinate.
     // In this example, Rectangle calls the getZoomPoint method and passes the zoomScale and the coordinates of the top-left corner as parameters to get the coordinates of the top-left corner after zooming
     // For width and height, simply multiply the original width and height by zoomScale
     zoom(zoomScale: number) {
       const leftTop = this.getZoomPoint(zoomScale, this.leftTop)
       this.update({
         x: leftTop.x,
         y: leftTop.y,
         width: this.width * zoomScale,
         height: this.height * zoomScale,
       })
     }
   }
   ```

 ##### Shape State

   - Different from the state of vue-stay-canvas, Shape itself also has a state field. When your Shape needs different drawing effects in different states, you can use the state field to control. For example, we need to implement a Rectangle. By default, the state is default, and the Rectangle draws a hollow rectangle. When the user moves the mouse over the Rectangle, we change the state of the Rectangle to hover, and when drawing, we set the color of the Rectangle to red. When the user moves the mouse out of the Rectangle, we change the state of the Rectangle to default and restore its color

   ```typescript
   // Define a custom Rectangle and control the drawing effects in different states through custom stateDrawFuncMap
   export class CustomRectangle extends Rectangle {
     constructor({ x, y, width, height, props = {} }: RectangleAttr) {
       super({ x, y, width, height, props })
       this.initStateDrawFuncMap()
     }

     initStateDrawFuncMap() {
       this.stateDrawFuncMap = {
         default: ({ context }) => {
           this.setColor(context, "white")
           context.strokeRect(this.x, this.y, this.width, this.height)
         },
         hover: ({ context }) => {
           this.setColor(context, "red")
           context.strokeRect(this.x, this.y, this.width, this.height)
         },
       }
     }
   }

   // Create a hover listener. When the mouse moves over the Rectangle, we change its state to hover. When the mouse moves out of the Rectangle, we change its state to default
   export const HoverListener: ListenerProps = {
     name: "hoverListener",
     event: "mousemove",
     callback: ({ e, tools: { getChildrenBySelector } }) => {
       const labels = getChildrenBySelector(".label")
       labels.forEach((label) => {
         let rectState = label.shape.contains(e.point) ? "hover" : "default"
         label.shape.switchState(rectState)
       })
     },
   }
   ```
   <video src="videos/state-map.mp4" controls="">
   </video>

 ##### Animation

   When creating custom Shapes, you need to implement the draw method. This method has a now timestamp parameter. You can use this parameter to create animation effects.
   In the above example, when hovering over the CustomRectangle, we hope the rectangle has a breathing effect where the border changes from white to red and back to white within 2 seconds instead of directly turning red. Then we can modify the hover drawing function in stateDrawFuncMap as follows:

   ```typescript
   ...
   hover: ({ context, now }) => {
     const c = Math.abs(
       Math.ceil((now % 1000) / 4) - 255 * (Math.floor((now % 10000) / 1000) % 2)
     )
     this.setColor(context, `rgb(255, ${c}, ${c})`)
     context.strokeRect(this.x, this.y, this.width, this.height)
   },
   ...
   ```

   <video src="videos/shape-anim.mp4" controls="">
   </video>

   You can achieve more rich animation effects in combination with [gsap](https://gsap.com/), [tween](https://github.com/tweenjs/tween.js), and other animation libraries

 ### Listener API

 ```typescript
 declare const DEFAULTSTATE = "default-state"

 interface ListenerProps {
   name: string // Listener name. You can use any string as the name, but it must be unique
   state?: string // Listener state. We will introduce it later. The default value is DEFAULTSTATE
   selector?: string // Listener selector. We will introduce it later
   event: string | string[] // Listener event. When this event is triggered, the callback function of the listener will be executed. When event is an array, any event triggers will execute the callback function of the listener. At the same time, the event name will be returned in e.name in the callback function
   sortBy?: SortChildrenMethodsValues | ChildSortFunction // Method for sorting the selected elements after the selector selects elements. We will introduce it later. The default value is SORT_CHILDREN_METHODS.AREA_ASC = area-asc, which sorts by area in ascending order. You can also customize the sorting function
   callback: (p: ActionCallbackProps) => {
     [key: string]: any
   } | void
 }

 // Custom element sorting method
 export type ChildSortFunction = (a: StayChild, b: StayChild) => number
 ```

 #### Selector

 In vue-stay-canvas, a very simple selector function is implemented, mainly used to filter elements by name and id. When we use appendChild, updateChild, etc., a `className` attribute needs to be provided, and the object returned by these tool functions will contain an `id` attribute. When defining a selector, you can select the corresponding element by adding a symbol `.` in front of the `className` attribute and a symbol `#` in front of the `id` attribute.

 ```typescript
 const child1 = appendChild({
   className: "label",
   ...
 })
 const child2 = appendChild({
   className: "label",
   ...
 })

 getChildrenBySelector(".label") // returns [child1, child2]
 getChildrenBySelector("#" + child1.id) // returns child1
 getChildrenBySelector("!.label") // returns []
 ```

 #### State

 In vue-stay-canvas, you can control the current state through the state attribute. This attribute is a string, and the default state is DEFAULTSTATE = "default-state". The concept of state comes from automatic state machines. By setting the state, you can flexibly control when the listener should be triggered. Imagine we want to achieve the following function:

 - By default, dragging on the canvas draws a rectangle based on the mouse.
 - After selecting this rectangle, dragging on this rectangle will move the rectangle.

 We can set up three listeners to achieve this function:

 - The first listener has the state attribute set to DEFAULTSTATE and listens for drag events. In the callback function, we implement the drawing function of the shape.
 - The second listener has the state attribute set to DEFAULTSTATE and listens for click events. In the callback function, if we detect that the user clicked on this drawn element, we change the current state to "selected". Otherwise, we change the state to DEFAULTSTATE.
 - The third listener has the state attribute set to "selected" and listens for drag events. In the callback function, we implement the moving function of the selected rectangle.

 You can perform some simple logical operations on the state field.

 #### Simple Logic Operations

 You can perform some very simple logical operations on certain attributes. Currently, the supported attributes include state and selector.

 ```typescript
 export const SUPPORT_LOGIC_OPRATOR = {
   AND: "&",
   OR: "|",
   NOT: "!",
 }

 const selector = ".A&#B" // select elements with className A and id B
 const selector = ".A&!#B" // select elements with className A and id not B
 const selector = "!.A" // select elements with className not A

 const state = "!selected" // when the state is not selected
 const state = "default-state|selected" // when the state is default-state or selected
 ```

 #### Event

 The event attribute accepts a string. You can pass an array of events in the eventList of StayCanvas to customize the event or directly override the predefined events. Events with the same name will be overridden. The specific method of customizing events will be introduced later.

 In vue-stay-canvas, the following events are predefined:

 - mousedown: Mouse button pressed down
 - mousemove: Mouse moved
 - mouseup: Mouse button released
 - keydown: Keyboard key pressed down
 - keyup: Keyboard key released
 - zoomin: Mouse wheel scrolled up
 - zoomout: Mouse wheel scrolled down
 - dragstart: Mouse left button pressed down
 - drag: Mouse left button pressed and moved and the mouse distance from the position where the mouse was pressed is greater than 10
 - dragend: Dragging ended
 - startmove: Ctrl key pressed and mouse left button pressed down
 - move: Ctrl key pressed and mouse left button pressed and moved
 - click: Clicked
 - redo: Ctrl + Shift + Z
 - undo: Ctrl + Z

 #### Listener Callback Function

 The callback function is the core function used to control user interactions on the canvas. This function is defined as follows:

 ```typescript
 // The parameter of this function is of type ActionCallbackProps. You can return a CallbackFuncMap object or return nothing.
 export type UserCallback = (p: ActionCallbackProps) => CallbackFuncMap<typeof p> | void

 export type CallbackFuncMap<T extends ActionCallbackProps> = {
   [key in T["e"]["name"]]: () => { [key: string]: any } | void | undefined
 }

 // ActionCallbackProps is as follows
 export interface ActionCallbackProps {
   originEvent: Event // Native event, this parameter is the event parameter passed when addEventListener callback is triggered
   e: ActionEvent // Event object defined in vue-stay-canvas, refer to ActionEvent type for specific information
   store: storeType // A global storage object of type Map
   stateStore: storeType // A storage object of type Map. The difference from store is that this object is cleared when the state changes
   composeStore: Record<string, any> // When we define a listener and pass an array to the event parameter, composeStore will be the same object for each event trigger
   tools: StayTools // StayTools object, which contains some utility functions. Detailed information will be introduced in StayTools.
   payload: Dict // The parameters passed when we manually call the trigger function
 }

 export interface ActionEvent {
   state: string // State when the event is triggered
   name: string // Event name
   x: number // The x-coordinate of the mouse relative to the canvas
   y: number // The y-coordinate of the mouse relative to the canvas
   point: Point // The coordinates of the mouse relative to the canvas
   target: StayChild // The element that triggered the event
   pressedKeys: Set<string> // The current pressed keys and mouse buttons. Mouse left button is mouse0, mouse right button is mouse1, mouse wheel is mouse2
   key: string | null // The keyboard key that triggered the event. When triggering the mouseup event, the key is not in pressedKeys, but is in key
   isMouseEvent: boolean // Whether it is a mouse event
   deltaX: number // The x-axis offset when the mouse wheel is scrolled
   deltaY: number // The y-axis offset when the mouse wheel is scrolled
   deltaZ: number // The z-axis offset when the mouse wheel is scrolled
 }

 // example 1
 // In this example, the callback function does not return anything. This listener switches the state of the rectangle based on whether the mouse is within the rectangle when the mouse is moved
 export const HoverListener: ListenerProps = {
   name: "hoverListener",
   event: "mousemove",
   callback: ({ e, tools: { getChildrenBySelector } }) => {
     const labels = getChildrenBySelector(".label")
     labels.forEach((label) => {
       let rectState = label.shape.contains(e.point) ? "hover" : "default"
       label.shape.switchState(rectState)
     })
   },
 }

 // example 2
 // In this example, the callback function returns a CallbackFuncMap object. Note that the event of this listener is an array, corresponding to the three keys returned by the callback function. Each key corresponds to a function that will be executed when dragstart, drag, and dragend events are triggered, respectively. The returned value will be merged into composeStore.

 // In the dragstart listener, we recorded dragStartPosition and dragChild and returned them. In the drag listener, we can get dragStartPosition and dragChild through composeStore to implement the drag function

 // In the dragend listener, we called the log function, which will take a snapshot of the current vue-stay-canvas. Later, we can use the undo/redo functions to implement undo/redo functionality

 const DragListener: ListenerProps = {
   name: "dragListener",
   event: ["dragstart", "drag", "dragend"],
   callback: ({ e, composeStore, tools: { appendChild, updateChild, log } }) => {
     return {
       dragstart: () => ({
         dragStartPosition: new Point(e.x, e.y),
         dragChild: appendChild({
           shape: new CustomRectangle({
             x: e.x,
             y: e.y,
             width: 0,
             height: 0,
             props: { color: "white" },
           }),
           className: "label",
         }),
       }),
       drag: () => {
         const { dragStartPosition, dragChild } = composeStore
         const x = Math.min(dragStartPosition.x, e.x)
         const y = Math.min(dragStartPosition.y, e.y)
         const width = Math.abs(dragStartPosition.x - e.x)
         const height = Math.abs(dragStartPosition.y - e.y)
         updateChild({
           child: dragChild,
           shape: dragChild.shape.update({ x, y, width, height }),
         })
       },
       dragend: () => {
         log()
       },
     }
   },
 }
 ```

 #### StayTools Utility Functions

 The StayTools object contains some utility functions, defined as follows:

 ```typescript
 export interface StayTools {
   createChild: <T extends Shape>(props: createChildProps<T>) => StayChild<T>
   appendChild: <T extends Shape>(props: createChildProps<T>) => StayChild<T>
   updateChild: (props: updateChildProps) => StayChild
   removeChild: (childId: string) => void
   getContainPointChildren: (props: getContainPointChildrenProps) => StayChild[]
   hasChild: (id: string) => boolean
   fix: () => void
   switchState: (state: string) => void
   getChildrenBySelector: (
     selector: string,
     sortBy?: SortChildrenMethodsValues | ChildSortFunction
   ) => StayChild[]
   getAvailableStates: (selector: string) => string[]
   changeCursor: (cursor: string) => void
   moveStart: () => void
   move: (offsetX: number, offsetY: number) => void
   zoom: (deltaY: number, center: SimplePoint) => void
   log: () => void
   redo: () => void
   undo: () => void
   triggerAction: (originEvent: Event, triggerEvents: Record<string, any>, payload: Dict) => void
   deleteListener: (name: string) => void
 }
 ```

 ##### Element Creation and Update

 - [`createChild`](#createchild) - Create a new element
 - [`appendChild`](#appendchild) - Create a new element and add it to the canvas
 - [`updateChild`](#updatechild) - Update the properties of an existing element
 - [`removeChild`](#removechild) - Remove an element from the canvas

 ##### Element Query and Judgment

 - [`getContainPointChildren`](#getcontainpointchildren) - Get all elements containing a certain point
 - [`hasChild`](#haschild) - Determine whether an element exists on the canvas
 - [`getChildrenBySelector`](#getchildrenbyselector) - Get elements by selector
 - [`getAvailableStates`](#getavailablestates) - Get all available states

 ##### State and View Control

 - [`fix`](#fix) - Adjust all elements' layers to the bottom layer
 - [`switchState`](#switchstate) - Switch the current state
 - [`changeCursor`](#changecursor) - Change the mouse pointer style
 - [`moveStart`](#movestart) - Start moving all elements
 - [`move`](#move) - Move all elements
 - [`zoom`](#zoom) - Zoom all elements

 ##### Snapshot Control

 - [`log`](#log) - Save the current canvas snapshot
 - [`redo`](#redo) - Move forward to the next snapshot
 - [`undo`](#undo) - Move back to the previous snapshot

 ##### Event Trigger

 - [`triggerAction`](#triggeraction) - Manually trigger an event
 - [`deleteListener`](#deletelistener) - Delete a listener

 ##### createChild

 The createChild function is used to create an element. This function accepts an object as a parameter. The parameter is defined as follows:

 ```typescript
 createChild: <T extends Shape>(props: createChildProps<T>) => StayChild<T>

 export interface createChildProps<T> {
   id?: string // The id of the element. If not specified, it will be generated automatically
   zIndex?: number // The zIndex of the element. This value affects the drawing order of the element on the canvas. The smaller the zIndex value, the more forward it is drawn. The default value is 1
   shape: T // The shape of the element. This value must be a subclass of Shape. vue-stay-canvas has built-in subclasses of Shape, and you can also customize your own Shape subclass. Detailed information will be introduced later
   className: string // The className of the element
   layer?: number // The layer where the element is located. This value is the index of the canvas layer where the element is located. 0 means the bottom layer. The larger the value, the closer to the top layer. You can also use negative numbers to indicate layers. -1 means the top layer, -2 means the next layer below the top layer, and so on. The default value is -1
 }

 // example:
 createChild({
   shape: new Rectangle({
     x: e.x,
     y: e.y,
     width: 0,
     height: 0,
     props: { color: "white" },
   }),
   className: "annotation",
 })
 ```

 ##### appendChild

 The appendChild function is used to create an element and add it directly to the canvas. This function accepts an object as a parameter. The parameter definition is the same as that of the createChild function.

 ```typescript
 // example
 appendChild({
   shape: new Rectangle({
     x: e.x,
     y: e.y,
     width: 0,
     height: 0,
     props: { color: "white" },
   }),
   className: "annotation",
 })
 ```

 ##### updateChild

 The updateChild function is used to update an element. This function accepts an object as a parameter. The parameters accepted by this function are different from those of the createChild function in that it requires a child object. This object can be obtained from the value returned by the appendChild or createChild function. In addition, other parameters are optional. The parameter definition is as follows:

 ```typescript
 export type updateChildProps<T = Shape> = {
   child: StayChild
 } & Partial<createChildProps<T>>

 // example
 updateChild({
   child,
   shape: child.shape.update({
     x,
     y,
     width,
     height,
   }),
 })
 ```

 ##### removeChild

 The removeChild function is used to delete an element. This function accepts a string parameter childId, which is the id of the element. It has no return value.

 ```typescript
 // example
 removeChild(image.id)
 ```

 ##### getContainPointChildren

 The getContainPointChildren function is used to get all elements containing a certain point. When using this function, you need to specify a selector to define the search range. The parameter definition is as follows:

 ```typescript
 export interface getContainPointChildrenProps {
   selector: string | string[] // The selector. This value can be a string or an array of strings. When it is an array of strings, it will return the union of all selector search results
   point: SimplePoint // The coordinates of the mouse relative to the canvas || interface SimplePoint {x: number y: number}
   returnFirst?: boolean | undefined // Whether to return only the first element. The default value is false
   sortBy?: SortChildrenMethodsValues | ChildSortFunction // The sorting method. The default value is SORT_CHILDREN_METHODS.AREA_ASC = area-asc. You can also pass in a function to customize the element sorting method
 }

 // The following are the built-in sorting methods
 export type SortChildrenMethodsValues = valueof<typeof SORT_CHILDREN_METHODS>
 export const SORT_CHILDREN_METHODS = {
   X_ASC: "x-asc", // Sort by X-axis in ascending order
   X_DESC: "x-desc", // Sort by X-axis in descending order
   Y_ASC: "y-asc", // Sort by Y-axis in ascending order
   Y_DESC: "y-desc", // Sort by Y-axis in descending order
   WIDTH_ASC: "width-asc", // Sort by width in ascending order
   WIDTH_DESC: "width-desc", // Sort by width in descending order
   HEIGHT_ASC: "height-asc", // Sort by height in ascending order
   HEIGHT_DESC: "height-desc", // Sort by height in descending order
   AREA_ASC: "area-asc", // Sort by area in ascending order
   AREA_DESC: "area-desc", // Sort by area in descending order
 }

 // example
 getContainPointChildren({
   point: new Point(100, 100),
   selector: ".annotation",
   sortBy: "area-asc",
 })
 ```

 ##### hasChild

 The hasChild function is used to determine whether an element exists on the canvas. This function accepts a string parameter childId, which is the id of the element. The return value is a boolean value, true means it exists, false means it does not exist.

 ```typescript
 // example
 hasChild(image.id)
 ```

 ##### fix

 The fix function is used to adjust the layers of all elements on the canvas to the bottom layer, which is equivalent to setting the layer of all elements to 0.

 ```typescript
 // example
 fix()
 ```

 ##### switchState

 The switchState function is used to switch the current state. This function accepts a string parameter state. After switching the state, the values in stateStore will be cleared.

 ```typescript
 // example
 switchState("state1")
 ```

 ##### getChildrenBySelector

 The getChildrenBySelector function is used to get elements found by the selector. Its selector and sortBy parameters are the same as those of the getContainPointChildren function. The return value is an array of StayChild.

 ```typescript
 // example
 getChildrenBySelector({
   selector: ".annotation",
   sortBy: (a, b) => a.shape.area - b.shape.area,
 })
 ```

 ##### getAvailableStates

 The getAvailableStates function is a utility function. This function accepts a string and returns all states that match the selector among the states that have appeared so far.

 ```typescript
 // Suppose there are currently state1, state2, state3, state4, state5, state6, state7, state8, state9, state10 among all registered listeners, of which state1, state2, state3, state4, state5 have been triggered.
 // Specifically, when the selector is "all-state", all states will be returned.
 getAvailableStates("all-state") // returns ["state1", "state2", "state3", "state4", "state5"]
 getAvailableStates("!state1") // returns ["state2", "state3", "state4", "state5"]
 getAvailableStates("all-state&!(state1|state2)") // returns ["state3", "state4", "state5"]
 ```

 ##### changeCursor

 The changeCursor function is used to change the mouse pointer style. This function accepts a string parameter cursor, which is the style of the mouse pointer. Refer to <https://developer.mozilla.org/zh-CN/docs/Web/CSS/cursor> for specific values.

 ```typescript
 // example
 changeCursor("pointer")
 ```

 ##### moveStart

 The moveStart function is used to start moving all elements on the canvas. Before calling the move function, you need to call this function to save the position before the move.

 ##### move

 The move function is used to move all elements on the canvas. offsetX and offsetY represent the offset of the horizontal and vertical coordinates relative to the start position, respectively.

 ```typescript
 // example
 // Suppose we need to implement the function of moving the entire canvas when dragging. The listener can be written as follows
 export const MoveListener: ListenerProps = {
   name: "moveListener",
   event: ["startmove", "move"],
   state: ALLSTATE,
   callback: ({ e, composeStore, tools: { moveStart, move } }) => {
     const eventMap = {
       startmove: () => {
         moveStart()
         return {
           startMovePoint: new Point(e.x, e.y),
         }
       },
       move: () => {
         const { startMovePoint } = composeStore
         if (!startMovePoint) {
           return
         }
         move(e.x - startMovePoint.x, e.y - startMovePoint.y)
       },
     }
     return eventMap[e.name as keyof typeof eventMap]()
   },
 }
 ```

 ##### zoom

 The zoom function is used to zoom all elements on the canvas. This function accepts two parameters. The first parameter is the zoom ratio, usually e.deltaY. The second parameter is the zoom center point. When we implement the function of zooming based on the mouse, this parameter is the position of the mouse.

 ```typescript
 // example
 // Suppose we need to implement the function of zooming the entire canvas when scrolling the mouse wheel. The listener can be written as follows
 export const ZoomListener: ListenerProps = {
   name: "zoomListener",
   event: ["zoomin", "zoomout"],
   state: ALLSTATE,
   callback: ({ e, tools: { zoom } }) => {
     zoom(e.deltaY, new Point(e.x, e.y))
   },
 }
 ```

 ##### log

 The log function saves the current canvas snapshot and pushes the current canvas snapshot into the stack. After executing this function, you can call the redo and undo functions to restore the previous snapshots.

 ##### redo

 Move forward to the next snapshot.

 ##### undo

 Move back to the previous snapshot.

 The redo function and the undo function are used to change the current canvas to the snapshot in the stack.

 We can modify some code in the initial example to simply understand this function.

 ```diff
 <script setup lang="ts">
 - import { ListenerProps, Point, Rectangle, StayCanvas } from "vue-stay-canvas"
 + import { ref } from 'vue'
 + import { ALLSTATE, Rectangle, type ListenerProps, StayCanvas } from 'vue-stay-canvas'
 + const stayCanvasRef = ref<InstanceType<typeof StayCanvas> | null>(null)

   const DragListener: ListenerProps = {
     name: "dragListener",
 -    event: ["dragstart", "drag"],
 -    callback: ({ e, composeStore, tools: { appendChild, updateChild } }) => {
 +    event: ["dragstart", "drag", "dragend"],
 +    callback: ({ e, composeStore, tools: { appendChild, updateChild, log } }) => {
       return {
         dragstart: () => ({
           dragStartPosition: e.point,
           dragChild: appendChild({
             shape: new Rectangle({
               x: e.x,
               y: e.y,
               width: 0,
               height: 0,
               props: { color: "red" },
             }),
             className: "annotation",
           }),
         }),
         drag: () => {
           const { dragStartPosition, dragChild } = composeStore
           const x = Math.min(dragStartPosition.x, e.x)
           const y = Math.min(dragStartPosition.y, e.y)
           const width = Math.abs(dragStartPosition.x - e.x)
           const height = Math.abs(dragStartPosition.y - e.y)
           updateChild({
             child: dragChild,
             shape: dragChild.shape.update({ x, y, width, height }),
           })
         },
 +        // Add dragend event. After dragging ends, we save the current canvas snapshot
 +        dragend: () => {
 +          log()
 +        },
       }
     },
   }

 +  // Add redo and undo listeners
 +  const undoListener: ListenerProps = {
 +    name: "undoListener",
 +    event: "undo",
 +    callback: ({ tools: { undo } }) => {
 +      undo()
 +    },
 +  }

 +  const redoListener: ListenerProps = {
 +    name: "redoListener",
 +    event: "redo",
 +    callback: ({ tools: { redo } }) => {
 +      redo()
 +    },
 +  }

 +  // Call the trigger method of vue-stay-canvas to trigger the listener
 +  function trigger(name: string) {
 +    if (!stayCanvasRef.value) {
 +      return
 +    }
 +    stayCanvasRef.value.trigger(name)
 +  }
 </script>
 <template>
 +     Add two buttons to trigger redo and undo events, respectively
 +     <button @click="() => trigger('redo')">redo</button>
 +     <button @click="() => trigger('undo')">undo</button>
 -      return <StayCanvas :width="500" :height="500" listenerList="[DragListener]" />
 +      <StayCanvas
 +        ref="stayCanvasRef"
 +        :width="500"
 +        :height="500"
 +        // Add the two newly added listeners to the listenerList
 +        :listenerList="[DragListener, undoListener, redoListener]"
 +      />
 </template>
 ```

 <video src="videos//redo-undo.mp4" controls="">
 </video>

 ##### triggerAction

 The triggerAction function is used to manually trigger events. Its effect is the same as calling trigger, but you need to manually construct the Event object and pass in the triggerEvents object.

 ```typescript
 type triggerEventsProps = { [key: string]: ActionEvent },
 ```

 ##### deleteListener

 The deleteListener function is used to delete a listener. This function accepts a string parameter listenerName, which is the name of the listener. This function will delete the listener. If the listener does not exist, no operation will be performed.

 ### Event API

 ```typescript
 type EventProps = {
   name: string
   trigger: valueof<typeof MOUSE_EVENTS> | valueof<typeof KEYBOARD_EVENTS>
   conditionCallback?: (props: UserConditionCallbackProps): boolean
   successCallback?: (props: UserSuccessCallbackProps) => void | EventProps
 }

 export const MOUSE_EVENTS = {
   MOUSE_DOWN: "mousedown", // Mouse button pressed down event type constant, used in mouse down event listeners.
   MOUSE_UP: "mouseup", // Mouse button released event type constant, used in mouse up event listeners.
   MOUSE_MOVE: "mousemove", // Mouse moved event type constant, used in mouse move event listeners.
   WHEEL: "wheel", // Mouse wheel event type constant, used in mouse wheel event listeners.
   CLICK: "click", // Mouse click event type constant, used in mouse click event listeners.
   DB_CLICK: "dblclick", // Mouse double click event type constant, used in mouse double click event listeners.
   CONTEXT_MENU: "contextmenu", // Mouse right click event type constant, used in mouse right click event listeners.
 } as const

 export const KEYBOARD_EVENTS = {
   KEY_DOWN: "keydown", // Keyboard key pressed down event type constant, used in keyboard down event listeners.
   KEY_UP: "keyup", // Keyboard key released event type constant, used in keyboard up event listeners.
 } as const
 ```

 Next, we will introduce the attributes of EventProps one by one.

 #### name

 The name attribute is used to identify the event. This attribute is a string. When there are two events with the same name, the latter will override the former.

 #### trigger

 The trigger indicates the trigger condition of the event. Currently, some values in MOUSE_EVENTS and KEYBOARD_EVENTS are supported. Refer to the above constant definitions for details.

 ##### Explanation

 - If we want to customize a move event for moving the entire canvas, the trigger condition for this event is that the user needs to press the Ctrl key on the keyboard and drag with the mouse left button pressed down. Then the value of this trigger should be "mousemove" because we need to know the position of the mouse when triggering this event and update the canvas based on the mouse position in real-time. Using "keydown" and "mousedown" is not appropriate because these two events are only triggered once. We need a continuously triggered event, so we need to use "mousemove".

 ```typescript
 const MoveEvent: EventProps = {
   name: "move",
   trigger: MOUSE_EVENTS.MOUSE_MOVE,
   conditionCallback: ({ e, store }) => {
     return e.pressedKeys.has("Control") && e.pressedKeys.has("mouse0")
   },
 }
 ```

 #### conditionCallback

 The conditionCallback attribute accepts a function. The parameter of this function meets the UserConditionCallbackProps type constraint. The e/store/stateStore parameters in the parameter are the same as those passed in the listener callback. [Listener-callback-function](#listener-callback-function). This function needs to return a boolean value. If it returns true, it means the event trigger condition is met. If it returns false, it means the event trigger condition is not met.

 ```typescript
 export interface UserConditionCallbackFunction {
   (props: UserConditionCallbackProps): boolean
 }

 export interface UserConditionCallbackProps {
   e: ActionEvent
   store: storeType
   stateStore: storeType
 }
 ```

 conditionCallback is an optional parameter. When we do not pass this parameter, it means that the event will be triggered when the trigger condition is met. For example, if we need to define a mouse down event, we can define it as follows:

 ```typescript
 const MouseDownEvent: EventProps = {
   name: "mousedown",
   trigger: MOUSE_EVENTS.MOUSE_DOWN,
 }
 ```

 #### successCallback

 The successCallback attribute accepts a function. The parameter of this function meets the UserSuccessCallbackProps type constraint. The e/store/stateStore parameters in the parameter are the same as those passed in the listener callback. [Listener-callback-function](#listener-callback-function). At the same time, there is an additional deleteEvent function in the parameter, which is used to delete events. This function also accepts an optional return value. When the return value is of type EventProps, the returned event will be registered after this event is triggered.

 This function can be very useful in some cases. One scenario is that when we need to define a set of drag events, one approach is to define three events: start drag, drag in progress, and end drag. However, we hope that the drag in progress event will only take effect after the start drag event is triggered. This way, we can avoid the situation where the mouse is pressed outside the canvas and then moved into the canvas, directly triggering the drag event. In this case, we cannot get the mouse position when the drag started. We also hope to trigger the end drag event only after the drag event is triggered. Imagine if the user directly clicks in the canvas, we will first trigger the start drag event, then skip the trigger of the drag event, and directly trigger the end drag event. This way, we may get unexpected results in some cases.

 Here is a method for registering drag events:

 ```typescript
 // Define the end drag event
 const DragEndEvent: EventProps = {
   name: "dragend", // Event name
   trigger: MOUSE_EVENTS.MOUSE_UP, // Trigger condition, mouse released
   successCallback: ({ store, deleteEvent }) => {
     deleteEvent("drag") // Delete the drag in progress event in the success callback
     deleteEvent("dragend") // Delete the end drag event itself
     store.set("dragging", false) // Update the state to indicate the end of dragging
   },
 }

 // Define the drag in progress event
 const DragEvent: EventProps = {
   name: "drag", // Event name
   trigger: MOUSE_EVENTS.MOUSE_MOVE, // Trigger condition, mouse moved
   conditionCallback: ({ e, store }) => {
     const dragStartPosition: Point = store.get("dragStartPosition")
     return (
       e.pressedKeys.has("mouse0") && // Check if the mouse left button is pressed
       (dragStartPosition.distance(e.point) >= 10 || store.get("dragging")) // Check if the mouse movement distance is sufficient or the dragging state is already in progress
     )
   },
   successCallback: ({ store }) => {
     store.set("dragging", true) // Set the state to dragging in progress
     return DragEndEvent // Return the end drag event for registration
   },
 }

 // Define the start drag event
 export const DragStartEvent: EventProps = {
   name: "dragstart", // Event name
   trigger: MOUSE_EVENTS.MOUSE_DOWN, // Trigger condition, mouse pressed down
   conditionCallback: ({ e }) => {
     return e.pressedKeys.has("mouse0") // Mouse left button pressed
   },
   successCallback: ({ e, store }) => {
     store.set("dragStartPosition", e.point) // Store the mouse position when the drag started
     return DragEvent // Return the drag in progress event for registration
   },
 }

 // The event registration list only contains the start drag event. Other events are dynamically registered through callbacks.
 const eventList = [DragStartEvent]
 ```

 `DragStartEvent`: Defines a start drag event. When the mouse left button is pressed down, it triggers. In the success callback, it sets the drag start position and returns the DragEvent object to register this event and start tracking the drag movement.

 `DragEvent`: Defines the drag in progress event. This event is triggered when the mouse moves but only when certain conditions are met (mouse left button pressed and the movement distance exceeds 10 pixels or the dragging state is already in progress). Its success callback sets the state to dragging in progress and returns the DragEndEvent object for registration.

 `DragEndEvent`: Defines the end drag event. This event is triggered when the mouse button is released. In its success callback, it clears all drag-related events (including the in-progress and self-ending events) and sets the dragging state to not in progress.

 ### trigger Function API
 Before using the trigger function, you need to use ref to get the reference of vue-stay-canvas.

 You can use the trigger function to manually trigger events. Sometimes you may need to trigger events outside the canvas, such as changing the state of the entire canvas, loading some data, saving some data, etc. You may want to trigger it when the user clicks a button outside the canvas or automatically. Then using the trigger function can achieve this.

 This function accepts two parameters. The first parameter is the event name, and the second parameter is the event-carrying parameter. This parameter will be passed to the payload parameter in the [Listener-callback-function](#listener-callback-function).

 ```typescript
 export type Dict = Record<string, any>
 export type TriggerFunction = (name: string, payload?: Dict) => void

 // example:
 const stayCanvasRef = ref<InstanceType<typeof StayCanvas> | null>(null)
 <StayCanvas
   ref="stayCanvasRef"
   ...
 />

 export const StateChangeListener: ListenerProps = {
   name: "changeState",
   event: "changeState",
   state: ALLSTATE,
   callback: ({ tools: { switchState }, payload }) => {
     switchState(payload.state)
   },
 }

 function trigger(name: string, payload?: Dict) {
   if (!stayCanvasRef.value) {
     return
   }
   stayCanvasRef.value.trigger(name, payload)
 }

 // Call trigger externally to trigger the changeState event and execute the callback function corresponding to StateChangeListener
 trigger("changeState", { state: "draw" })
 ```
