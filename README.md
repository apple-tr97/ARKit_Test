# Demo ARKit 
## Crear una app con ARKit

# Indice
- [Tutorial en Youtube](https://www.youtube.com/watch?v=jj253939vJ_Kc&feature=youtu.be)
- [App](#ARKit)

## **ARKit**

1. Inciamos Xcode
2. **"Create a new Xcode Proyect"**
3. Seleccionamos pestaña **iOS**
4. En la sección Application elegimos **"Augmented Reality App"**
5. Nombramos el proyecto 
6. Guardamos y creamos el proyecto
7. Hasta este punto podriamos ya probar la app creada y nos mostraria un avion
![alt text](https://github.com/apple-tr97/ARKit_Test/blob/master/IMG_0746.PNG)
Sin embargo con ARKit tambien se puede escanear el ambiente y detectar planos con el siguiente codigo:
8. Creamos un nuevo archivo en swift llamado Plane.swift para visualizar los planos que se detecten
```swift
import ARKit

extension UIColor {
    static let planeColor = UIColor(named: "planeColor")!
}
//Clase para visualizar planos utilizando nodos
class Plane: SCNNode {
   
    let meshNode: SCNNode
    let extentNode: SCNNode
    var classificationNode: SCNNode?
    
    init(anchor: ARPlaneAnchor, in sceneView: ARSCNView) {
        
        #if targetEnvironment(simulator)
        #error("ARKit is not supported in iOS Simulator. Connect a physical iOS device and select it as your Xcode run destination, or select Generic iOS Device as a build-only destination.")
        #else

        
        // Crear la "malla" o mesh para visualizar la forma del plano
        guard let meshGeometry = ARSCNPlaneGeometry(device: sceneView.device!)
            else { fatalError("Can't create plane geometry") }
        meshGeometry.update(from: anchor.geometry)
        meshNode = SCNNode(geometry: meshGeometry)
        
      
        // Crear un nodo para visualizar los limites del plano detectado
        let extentPlane: SCNPlane = SCNPlane(width: CGFloat(anchor.extent.x), height: CGFloat(anchor.extent.z))
        extentNode = SCNNode(geometry: extentPlane)
        extentNode.simdPosition = anchor.center
        
        extentNode.eulerAngles.x = -.pi / 2

        super.init()

        self.setupMeshVisualStyle()
        self.setupExtentVisualStyle()

        
        // Añadir la extension del plano y su forma como un nodo para que se visualice 
        addChildNode(meshNode)
        addChildNode(extentNode)
        
        // Desplegar el plano si es que es soportado por el dispositivo(iOS 12 en adelante)
        if #available(iOS 12.0, *), ARPlaneAnchor.isClassificationSupported {
            let classification = anchor.classification.description
            let textNode = self.makeTextNode(classification)
            classificationNode = textNode
            textNode.centerAlign()
            extentNode.addChildNode(textNode)
        }
        #endif
    }
    
    required init?(coder aDecoder: NSCoder) {
        fatalError("init(coder:) has not been implemented")
    }
    
    private func setupMeshVisualStyle() {      
        // Hacer el plano semi transparente para poder ver el "Mundo Real"
        meshNode.opacity = 0.25
       
        guard let material = meshNode.geometry?.firstMaterial
            else { fatalError("ARSCNPlaneGeometry always has one material") }
        material.diffuse.contents = UIColor.planeColor
    }
    
    private func setupExtentVisualStyle() {
        // Hacer el plano semi transparente para poder ver el "Mundo Real"
        extentNode.opacity = 0.6

        guard let material = extentNode.geometry?.firstMaterial
            else { fatalError("SCNPlane always has one material") }
        
        material.diffuse.contents = UIColor.planeColor
      
    }
    
    
```
