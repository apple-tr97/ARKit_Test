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
9. Crearemos un nuevo archivo en swift llamado Utilities.swift para clasificar lo que detecte ARKit y sus limites
``` swift
import ARKit

@available(iOS 12.0, *)
extension ARPlaneAnchor.Classification {
    var description: String {
        switch self {
        case .wall:
            return "Wall"
        case .floor:
            return "Floor"
        case .ceiling:
            return "Ceiling"
        case .table:
            return "Table"
        case .seat:
            return "Seat"
        case .none(.unknown):
            return "Unknown"
        default:
            return ""
        }
    }
}

extension SCNNode {
    func centerAlign() {
        let (min, max) = boundingBox
        let extents = float3(max) - float3(min)
        simdPivot = float4x4(translation: ((extents / 2) + float3(min)))
    }
}

extension float4x4 {
    init(translation vector: float3) {
        self.init(float4(1, 0, 0, 0),
                  float4(0, 1, 0, 0),
                  float4(0, 0, 1, 0),
                  float4(vector.x, vector.y, vector.z, 1))
    }
}
```
10. Por ultimo en el viewController.swift utilizaremos ARSession el cual es el encargado de detertar los planos en ARKit
11. Eliminaremos el codigo por defeccto que viene en el viewcontroller
12. Añadir el sigueinte codigo:
``` swift
mport UIKit
import SceneKit
import ARKit

class ViewController: UIViewController, ARSCNViewDelegate, ARSessionDelegate {
    
    @IBOutlet weak var sceneView: ARSCNView!
    
    //Iniciar ARSession
    override func viewDidAppear(_ animated: Bool) {
        super.viewDidAppear(animated)

        //Iniciar una sesion AR inicializando la configuracion de la camara, posicion y orientacion del dispositivo y detección de planos
        let configuration = ARWorldTrackingConfiguration()
        configuration.planeDetection = [.horizontal, .vertical]
        sceneView.session.run(configuration)

     
        // Delegate para hacer seguimineto a los planos creados
        sceneView.session.delegate = self
              
        //Impedir el bloqueo de pantalla por no detectar interacciones con el usuario
        UIApplication.shared.isIdleTimerDisabled = true
        
        // Visualizar estadisticas como los frames
        sceneView.showsStatistics = true
    }

    override func viewWillDisappear(_ animated: Bool) {
        super.viewWillDisappear(animated)

        // Pausar sesion de ARSession
        sceneView.session.pause()
    }

    // Despligar AR
    func renderer(_ renderer: SCNSceneRenderer, didAdd node: SCNNode, for anchor: ARAnchor) {
        // Place content only for anchors found by plane detection.
        // Desplegar las "Anclas" de los planos detectados
        guard let planeAnchor = anchor as? ARPlaneAnchor else { return }
        
        //Crear el obheto para visualizar su forma y extensión 
        let plane = Plane(anchor: planeAnchor, in: sceneView)
        
        // Agregar la visualizacion al nodo de ARKit para darle seguimiento 
        node.addChildNode(plane)
    }

    func renderer(_ renderer: SCNSceneRenderer, didUpdate node: SCNNode, for anchor: ARAnchor) {
        // Actualizar anclas y nodos detectados
        guard let planeAnchor = anchor as? ARPlaneAnchor,
            let plane = node.childNodes.first as? Plane
            else { return }
        
        // Actualizar ARSCNPlaneGeometry a las nuevas anclas del plano
        if let planeGeometry = plane.meshNode.geometry as? ARSCNPlaneGeometry {
            planeGeometry.update(from: planeAnchor.geometry)
        }

        // Actualizar extension del plano a los nuevo limites detectados
        if let extentGeometry = plane.extentNode.geometry as? SCNPlane {
            extentGeometry.width = CGFloat(planeAnchor.extent.x)
            extentGeometry.height = CGFloat(planeAnchor.extent.z)
            plane.extentNode.simdPosition = planeAnchor.center
        }
        
        // Actualizar la clasificacion del plano 
        if #available(iOS 12.0, *),
            let classificationNode = plane.classificationNode,
            let classificationGeometry = classificationNode.geometry as? SCNText {
            let currentClassification = planeAnchor.classification.description
            if let oldClassification = classificationGeometry.string as? String, oldClassification != currentClassification {
                classificationGeometry.string = currentClassification
                classificationNode.centerAlign()
            }
        }
        
    }

    
    func sessionInterruptionEnded(_ session: ARSession) {
        // Reiniciar el seguimineto
        resetTracking()
    }
    
    
    private func resetTracking() {
        let configuration = ARWorldTrackingConfiguration()
        configuration.planeDetection = [.horizontal, .vertical]
        sceneView.session.run(configuration, options: [.resetTracking, .removeExistingAnchors])
    }
}
