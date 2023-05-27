<template>
    <div>
        <canvas ref="bjsCanvas" width="500" height="500" />
    </div>
</template>
  
<script setup lang="ts">
import { ref, onMounted } from "vue";
import { Engine, Scene, FollowCamera, PointerEventTypes, Vector3, MeshBuilder, StandardMaterial, Color3, PointLight, HemisphericLight, Animation } from "@babylonjs/core";

// 定义一个ref变量来获取画布元素
const bjsCanvas = ref<HTMLCanvasElement | null>(null);

// 在组件挂载后执行创建场景的函数
onMounted(createScene);

// 创建场景的函数
function createScene() {
    if (bjsCanvas.value) {
        // 创建一个引擎对象并绑定画布元素
        const engine = new Engine(bjsCanvas.value);
        const scene = new Scene(engine);

        // 创建光源
        new PointLight("light1", Vector3.Up(), scene);
        new HemisphericLight("light2", Vector3.Up(), scene);
        // new HemisphericLight("light3", Vector3.Down(), scene);

        // 创建一个黄色球作为小鸟
        const bird = MeshBuilder.CreateSphere("bird", {}, scene);
        const material = new StandardMaterial("bird-material", scene);
        material.diffuseColor = Color3.Yellow();
        bird.material = material;

        // 创建一个跟随摄像机，跟随小鸟
        const camera = new FollowCamera("FollowCamera", new Vector3(0, 10, -10), scene);
        camera.lockedTarget = bird;
        camera.radius = 10;
        camera.heightOffset = 4;
        camera.rotationOffset = 180;
        camera.cameraAcceleration = 0.1;
        camera.maxCameraSpeed = 5;
        camera.attachControl(bjsCanvas.value, true);

        // 创建两个会不断向右延伸的平面分别作为天空和地面
        const sky = MeshBuilder.CreateGround("sky", { width: 100, height: 100 }, scene);
        const materialSky = new StandardMaterial("sky-material", scene);
        materialSky.diffuseColor = Color3.Blue();
        sky.material = materialSky;
        sky.position.y = 5;
        const ground2 = MeshBuilder.CreateGround("ground2", { width: 100, height: 100 }, scene);
        const materialGreen2 = new StandardMaterial("ground-material2", scene);
        materialGreen2.diffuseColor = Color3.Green();
        ground2.material = materialGreen2;
        ground2.position.y = -5;




        //创建一个点击事件，点击小鸟的时候，让小鸟上升
        scene.onPointerObservable.add((pointerInfo) => {
            switch (pointerInfo.type) {
                case PointerEventTypes.POINTERDOWN:
                    bird.animations = [];
                    const animation = new Animation("bird-animation", "position.y", 30, Animation.ANIMATIONTYPE_FLOAT, Animation.ANIMATIONLOOPMODE_CYCLE);
                    const keys = [
                        { frame: 0, value: bird.position.y },
                        { frame: 10, value: bird.position.y + 1 },
                    ];
                    animation.setKeys(keys);
                    bird.animations = [animation];
                    scene.beginAnimation(bird, 0, 30, false);
                    break;
            }
        });
        //创建一个全局事件，让小鸟向某个方向移动并且会下落
        scene.onBeforeRenderObservable.add(() => {
            bird.position.y -= 0.03;
            bird.position.x += 0.01;
        });
        //创建一个全局事件，每过一段时间就会生成一个高度不会超过8的障碍物
        scene.onBeforeRenderObservable.add(() => {
            if (Math.random() > 0.99) {
                const obstacle = MeshBuilder.CreateBox("obstacle", { height: 5 }, scene);
                const materialObstacle = new StandardMaterial("obstacle-material", scene);
                materialObstacle.diffuseColor = Color3.Red();
                obstacle.material = materialObstacle;
                obstacle.position.x = 10;
                obstacle.position.y = -5 + obstacle.scaling.y / 2;
            }
        });


        //创建一个碰撞检测事件，当小鸟碰到地面、天空和障碍物的时候，提示游戏结束，解除其他的事件绑定
        scene.onBeforeRenderObservable.add(() => {
            if (bird.position.y < -4.5 || bird.position.y > 4.5) {
                alert("游戏结束");
                scene.onPointerObservable.clear();
                scene.onBeforeRenderObservable.clear();
            }
            scene.meshes.forEach((mesh) => {
                if (mesh.name === "obstacle") {
                    if (bird.intersectsMesh(mesh, false)) {
                        alert("游戏结束");
                        scene.onPointerObservable.clear();
                        scene.onBeforeRenderObservable.clear();
                    }
                }
            });
        });



        // 渲染循环
        engine.runRenderLoop(() => {
            scene.render();

            // TODO: 添加游戏逻辑和交互
            // 比如让小鸟上下飞动，生成障碍物，检测碰撞等
        });
    }
}
</script>
  