<html>
    <table style="margin-left: auto; margin-right: auto;">
        <tr>
            <td>
                <!--左侧内容-->
                左边
            </td>
            <td>
                <!--右侧内容-->
## Learning Discriminative Model Prediction for Tracking

**DiMP:** Goutam Bhat, Martin Danelljan, Luc Van Gool, Radu Timofte.<br />
  "Learning Discriminative Model Prediction for Tracking." ICCV (2019 **oral**). 
  [[paper](http://openaccess.thecvf.com/content_ICCV_2019/papers/Bhat_Learning_Discriminative_Model_Prediction_for_Tracking_ICCV_2019_paper.pdf)]
  [[code](https://github.com/visionml/pytracking)]

### Siamese 3个限制：

* 当inferring the model时，只利用target appearance，忽视背景信息（对区分相似性物体会有用）

* 学到的相似性度量（☞Siamese网络？？）对不在offline training set的目标可能不管用: poor generalization

* Siamese: no powerful model update strategy, 常用simple template averaging.

So, inferior robustness compared to other SOTA approaches.

### 作者的模型:

可以end-to-end model while maximizing the discriminative ability，achieved by:

* a steepest descent based methodology 计算最优的步长

* a module that can effectively initializes target model

Furthermore:

flexibility: by learning the discriminative learning loss itself.

            </td>
        </tr>
    </table>
</html>

