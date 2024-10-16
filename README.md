<details>
    <summary><h2>Uygulma Amacı</h2></summary>
  Bu kod, basit bir fizik tabanlı oyunun temelini oluşturuyor. Oyuncu, ekrana dokunarak kuşu sürükleyebilir ve serbest bırakarak kuşu hareket ettirebilir. Kuş, kutularla çarpıştığında skor artar ve kuş yavaşladığında oyun sıfırlanır. SpriteKit'in fizik motorunu kullanarak etkileşimleri yönetmek ve skor takibi yapmak için temel bir yapı sunar
  </details> 


  <details>
    <summary><h2>SpriteNode ve Değişkenlerin Tanımlanması</h2></summary>
    bird: Oyundaki kuşu temsil eden sprite düğümü.
    box1 - box5: Beş farklı kutuyu temsil eden sprite düğümleri.
    gameStared: Oyunun başladığını belirten bir bayrak.
    originalPosition: Kuşun orijinal konumunu saklar, kuşu sıfırlamak için kullanılır.
    score: Oyuncunun skorunu tutar.
    scoreLabel: Skoru ekranda göstermek için kullanılan etiket düğümü.

    
    ```
    var bird = SKSpriteNode()
    var box1 = SKSpriteNode()
    var box2 = SKSpriteNode()
    var box3 = SKSpriteNode()
    var box4 = SKSpriteNode()
    var box5 = SKSpriteNode()

    var gameStared = false
    var originalPosition: CGPoint?

    var score = 0
    var scoreLabel = SKLabelNode()



    ```
  </details> 




<details>
    <summary><h2>Sahne Yüklendiğinde (didMove) Yapılan Ayarlar</h2></summary>
    Fizik Sınırları ve Ölçekleme , Kuşun Ayarlanması , Kutuların Ayarlanması , Skor Etiketinin Ayarlanması

    
    ```
    override func didMove(to view: SKView) {
    // Fizik sınırları
    self.physicsBody = SKPhysicsBody(edgeLoopFrom: frame)
    self.scene?.scaleMode = .aspectFit
    self.physicsWorld.contactDelegate = self

    // Kuş Ayarları
    bird = childNode(withName: "bird") as! SKSpriteNode
    let birdTexture = SKTexture(imageNamed: "bird")
    bird.physicsBody = SKPhysicsBody(circleOfRadius: birdTexture.size().height / 19 )
    bird.physicsBody?.affectedByGravity = false
    bird.physicsBody?.isDynamic = true
    bird.physicsBody?.mass = 0.1
    originalPosition = bird.position
    bird.physicsBody?.contactTestBitMask = ColliderTyp.bird.rawValue
    bird.physicsBody?.categoryBitMask = ColliderTyp.bird.rawValue
    bird.physicsBody?.collisionBitMask = ColliderTyp.box.rawValue

    // Kutuların Ayarları
    let boxTexture = SKTexture(imageNamed: "brick")
    let size = CGSize(width: boxTexture.size().width / 7, height: boxTexture.size().height / 7)

    // Her bir kutu için fizik ayarları
    [box1, box2, box3, box4, box5].enumerated().forEach { index, box in
        box = childNode(withName: "box\(index + 1)") as! SKSpriteNode
        box.physicsBody = SKPhysicsBody(rectangleOf: size)
        box.physicsBody?.isDynamic = true
        box.physicsBody?.affectedByGravity = true
        box.physicsBody?.allowsRotation = true
        box.physicsBody?.mass = 0.3
        box.physicsBody?.collisionBitMask = ColliderTyp.bird.rawValue
    }

    // Skor Etiketi Ayarları
    scoreLabel.fontName = "Helvetica"
    scoreLabel.fontSize = 60
    scoreLabel.text = "0"
    scoreLabel.position = CGPoint(x: 0, y: self.frame.height / 4)
    scoreLabel.zPosition = 2
    self.addChild(scoreLabel)
    }




    ```
  </details>
  <details>
    <summary><h2>Çarpışma Başlangıcı (didBegin)</h2></summary>
    didBegin: İki fiziksel gövde çarpıştığında çağrılır.
    contact.bodyA ve contact.bodyB: Çarpışan iki gövdeyi temsil eder.
     Koşul: Çarpışan gövdelerden en az biri kuşsa (yani collisionBitMask değeri ColliderTyp.bird.rawValue ise), skor bir artırılır ve etiket güncellenir.

    
    ```
    func didBegin(_ contact: SKPhysicsContact) {
    if contact.bodyA.collisionBitMask == ColliderTyp.bird.rawValue || contact.bodyB.collisionBitMask == ColliderTyp.bird.rawValue {
        score += 1
        scoreLabel.text = String(score)
    }
    }



    ```
  </details> 
  <details>
    <summary><h2>Dokunuş Olayları</h2></summary>
    Kontrol: Oyun henüz başlamamışsa (gameStared == false).
Dokunulan Nokta: İlk dokunuş alınır ve dokunuş noktasındaki tüm düğümler (nodes(at:)) kontrol edilir.
Koşul: Eğer dokunuş yapılan düğüm kuşsa, kuşun pozisyonu dokunuş noktasına taşınır.
Amaç: Oyuncu kuşu dokunarak sürüklediğinde, kuşun konumunu güncellemek.

    
    ```
    override func touchesBegan(_ touches: Set<UITouch>, with event: UIEvent?) {
    if gameStared == false {
        if let touch = touches.first {
            let touchLocation = touch.location(in: self)
            let touchNodes = nodes(at: touchLocation)
            
            if touchNodes.isEmpty == false {
                for node in touchNodes {
                    if let sprite = node as? SKSpriteNode {
                        if sprite == bird {
                            bird.position = touchLocation
                        }
                    }
                }
            }
        }
    }
    }




    ```
  </details>
  <details>
    <summary><h2>Dokunuş Hareket Ettirildiğinde (touchesMoved)</h2></summary>
    Benzerlik: touchesBegan ile aynı şekilde çalışır.
    Fark: Dokunuş hareket ettirildiğinde, kuşun pozisyonu sürekli olarak güncellenir.
    Amaç: Oyuncu kuşu sürüklediğinde, kuşun konumunu güncel tutmak.

    
    ```
    override func touchesMoved(_ touches: Set<UITouch>, with event: UIEvent?) {
    if gameStared == false {
        if let touch = touches.first {
            let touchLocation = touch.location(in: self)
            let touchNodes = nodes(at: touchLocation)
            
            if touchNodes.isEmpty == false {
                for node in touchNodes {
                    if let sprite = node as? SKSpriteNode {
                        if sprite == bird {
                            bird.position = touchLocation
                        }
                    }
                }
            }
        }
    }
     }





    ```
  </details>
  <details>
    <summary><h2>Dokunuş Bittiğinde (touchesEnded)</h2></summary>
    Kontrol: Oyun henüz başlamamışsa.
    Dokunuş Sonu: İlk dokunuş alınır ve dokunuş noktasındaki düğümler kontrol edilir.
    Koşul: Eğer dokunuş bitişiğinde kuş dokunulmuşsa:
    dx ve dy Hesaplaması: Kuşun orijinal konumundan dokunuş konumuna olan farkın negatifini alır. Bu, kuşun çekilme yönünü belirler.
    Impulse Uygulama: Kuşun fizik gövdesine bir impuls (itme) uygulanır, böylece kuş serbestçe hareket etmeye başlar.
    affectedByGravity: Kuşun artık yerçekiminden etkilenmesini sağlar.
    gameStared: Oyunun başladığını belirtmek için true olarak ayarlanır.

    
    ```
    override func touchesEnded(_ touches: Set<UITouch>, with event: UIEvent?) {
    if gameStared == false {
        if let touch = touches.first {
            let touchLocation = touch.location(in: self)
            let touchNodes = nodes(at: touchLocation)
            
            if touchNodes.isEmpty == false {
                for node in touchNodes {
                    if let sprite = node as? SKSpriteNode {
                        if sprite == bird {
                            let dx = -(touchLocation.x - originalPosition!.x)
                            let dy = -(touchLocation.y - originalPosition!.y)
                            
                            let impulse = CGVector(dx: dx, dy: dy)
                            bird.physicsBody?.applyImpulse(impulse)
                            bird.physicsBody?.affectedByGravity = true
                            gameStared = true
                        }
                    }
                }
            }
        }
    }
    }






    ```
  </details>
  <details>
    <summary><h2>Oyunun Güncellenmesi (update)</h2></summary>
    Fonksiyon: Her kare güncellendiğinde (update), bu fonksiyon çağrılır.
    Kontrol:
    Hız Kontrolü: Kuşun hem x hem de y yönlerindeki hızının (velocity.dx ve velocity.dy) 0.1'den küçük olup olmadığını kontrol eder.
    Açısal Hız Kontrolü: Kuşun dönme hızının (angularVelocity) 0.1'den küçük olup olmadığını kontrol eder.
    gameStared: Oyunun başladığını doğrular.
   Eylemler:
   Yerçekimini Kapatma: Kuşun artık yerçekiminden etkilenmemesini sağlar.
   Hızı Sıfırlama: Kuşun hızını ve dönme hızını sıfırlar.
   Z Pozisyonunu Ayarlama: Kuşun z pozisyonunu 1 olarak ayarlar.
   Kuşu Orijinal Konuma Getirme: Kuşun pozisyonunu başlangıç konumuna sıfırlar.
   Skoru Sıfırlama: Skoru 0 yapar ve etiketi günceller.
   gameStared: Oyunun henüz başlamamış olduğunu belirtir.
   Amaç: Kuş durduğunda (yani çok yavaşladığında), oyunu sıfırlamak ve kuşu tekrar başlangıç konumuna getirmek.

    
    ```
    override func update(_ currentTime: TimeInterval) {
    if let birdPhysicsBody = bird.physicsBody {
        if birdPhysicsBody.velocity.dx <= 0.1 && birdPhysicsBody.velocity.dy <= 0.1 &&
            birdPhysicsBody.angularVelocity <= 0.1 && gameStared == true {
            bird.physicsBody?.affectedByGravity = false
            bird.physicsBody?.velocity = CGVector(dx: 0, dy: 0)
            bird.physicsBody?.angularVelocity = 0
            bird.zPosition = 1
            bird.position = originalPosition!
            
            score = 0
            scoreLabel.text = String(score)
            gameStared = false
        }
    }
    }







    ```
  </details>


  
  
  
<details>
    <summary><h2>Uygulama Görselleri </h2></summary>
    
    
 <table style="width: 100%;">
    <tr>
        <td style="text-align: center; width: 16.67%;">
            <h4 style="font-size: 14px;">Görüntü İşleme Sonuçları 1 </h4>
            <img src="https://github.com/user-attachments/assets/03ffaf10-3c0d-4217-9b06-2e4e41532c96" style="width: 100%; height: auto;">
        </td>
    </tr>
</table>
  </details> 
