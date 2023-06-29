<script>
// const detectInput = async () => {
//   const input = document.getElementById("previewImage");
//   const rect = input.getBoundingClientRect();
//   console.log("myVideo:" + rect.left, rect.top);
//   // console.log(input);
//   const detection = await faceapi
//     .detectAllFaces(input)
//     .withFaceExpressions()
//     .withAgeAndGender();
//   // console.log("detecction:", detection);
//   inputImgDataRef.value = detection;
//   inputImgDataRef.value.map((data, index) => {
//     let maxPro = 0;
//     let maxEmo = "";
//     for (const emo in data.expressions) {
//       console.log(
//         "current emo",
//         emo,
//         "maxPro:",
//         maxPro,
//         "curPro",
//         data.expressions[emo]
//       );
//       if (data.expressions[emo] > maxPro) {
//         maxPro = data.expressions[emo];
//         maxEmo = emo;
//       }
//     }
//     inputImgDataRef.value[index].finalExpressions = maxEmo;
//   });
//   console.log(dataRef.value);
// };
</script>
