```
@Data
@AllArgsConstructor
@NoArgsConstructor
public class Car {
    private String brand;
    private int price;
}
```



``` public class JsonTest {

        private final ObjectMapper objectMapper = new ObjectMapper();

        /**
         * JSON 객체 변환 에 대한 Test
         * */
        @Test
        public void json_바이트배열로_자바객체_변환()throws IOException {
            String carJson = "{\"brand\" : \"mercedes\", \"price\" : 100 }";
            byte[] bytes = carJson.getBytes("UTF-8");

            Car car = objectMapper.readValue(bytes, Car.class);

            System.out.println(car);
        }

        @Test
        public void json_바이트배열로_JsonNode객체_변환()throws IOException {
            String carJson = "{\"brand\" : \"mercedes\", \"price\" : 100 }";
            byte[] bytes = carJson.getBytes("UTF-8");

            JsonNode jnode = objectMapper.readTree(bytes);
            System.out.println(jnode);
        }

        @Test
        public void json_입력스트림으로_자바객체_변환()throws IOException {
            InputStream is = new FileInputStream("src\\data\\car.json");

            Car car = objectMapper.readValue(is, Car.class);

            System.out.println(car);
        }

        @Test
        public void json_입력스트림으로_JsonNode객체_변환()throws IOException {
            InputStream is = new FileInputStream("src\\data\\car.json");

            JsonNode jnode = objectMapper.readTree(is);
            System.out.println(jnode);
        }

        @Test
        public void json_파일로_자바객체_List_변환()throws IOException {
            File file = new File("src\\data\\cars.json");
            List<Car> cars = objectMapper.readValue(file, new TypeReference<List<Car>>() {
            });

            System.out.println(cars);
        }

        @Test
        public void json_파일로_JsonNode객체_변환()throws IOException {
            File file = new File("src\\data\\cars.json");

            JsonNode jnode = objectMapper.readTree(file);
            System.out.println(jnode);
        }

        @Test
        public void json값을_JsonParser로_역직렬화하여_TypeReference_클래스로_변환()throws IOException {
            String jsonArray = "[{\"brand\" : \"mercedes\", \"price\" : 100 }, {\"brand\" : \"Kia\", \"price\" : 60 }]";
            JsonParser jp = new JsonFactory().createParser(jsonArray);

            List<Car> cars = objectMapper.readValue(jp, new TypeReference<List<Car>>() {});

            System.out.println(cars);
        }

        @Test
        public void json값을_JsonParser로_역직렬화하여_TypeReference_클래스_JsonNode로_변환()throws IOException {
            String jsonArray = "[{\"brand\" : \"mercedes\", \"price\" : 100 }, {\"brand\" : \"Kia\", \"price\" : 60 }]";
            JsonParser jp = new JsonFactory().createParser(jsonArray);

            JsonNode jnode = objectMapper.readTree(jp);

            System.out.println(jnode);
        }

        @Test
        public void json배열로_자바객체_List_변환()throws IOException {
            String jsonArray = "[{\"brand\" : \"mercedes\", \"price\" : 100 }, {\"brand\" : \"Kia\", \"price\" : 60 }]";

            List<Car> cars = objectMapper.readValue(jsonArray, new TypeReference<List<Car>>() {});

            System.out.println(cars);
        }

        @Test
        public void json배열로_JsonNode_변환()throws IOException {
            String jsonArray = "[{\"brand\" : \"mercedes\", \"price\" : 100 }, {\"brand\" : \"Kia\", \"price\" : 60 }]";

            JsonNode jnode = objectMapper.readTree(jsonArray);

            System.out.println(jnode);
        }

        @Test
        public void json_URL객체로부터_자바객체_List_변환()throws IOException {
            URL url = new URL("file:src\\data\\cars.json");
            List<Car> cars = objectMapper.readValue(url, new TypeReference<List<Car>>() {});

            System.out.println(cars);
        }

        @Test
        public void json_URL객체로부터_JsonNode_변환()throws IOException {
            URL url = new URL("file:src\\data\\cars.json");

            JsonNode jnode = objectMapper.readTree(url);

            System.out.println(jnode);
        }

        /**
         * JSON 객체 출력 에 대한 Test
         * */

        @Test
        public void writeTree메서드를_이용하여_JSON_출력() throws IOException {
            JsonGenerator generator = objectMapper.getFactory().createGenerator(System.out);

            JsonNode jNode = objectMapper.createObjectNode();
            ((ObjectNode) jNode).put("brand", "Kia");
            ((ObjectNode) jNode).put("price", "60");

            objectMapper.writeTree(generator, jNode);
        }

        @Test
        public void 자바객체를_JSON으로_변환하여_파일로_출력() throws IOException {
            Car car = new Car("Kia", 60);
            objectMapper.writeValue(new File("src\\data\\target.json"), car);
        }

        @Test
        public void 자바_출력스트림으로_JSON_출력() throws IOException {
            Car car = new Car("Kia", 60);

            objectMapper.writeValue(System.out, car);
        }

        @Test
        public void 자바객체를_JSON형태의_바이트_행렬로_출력() throws IOException {
            Car car = new Car("Kia", 60);

            byte[] bytes = objectMapper.writeValueAsBytes(car);

            System.out.print(new String(bytes));
        }

        /**
         * JsonNode 사용법
         * */

        @Test
        public void json노드를_지정된_as형태로_출력() throws JsonProcessingException {
            String carJson = "{ \"brand\" : \"Kia\" , \"price\" : 60}";
            JsonNode jNode = objectMapper.readTree(carJson);

            String key = jNode.get("brand").asText();
            Integer valueInt = jNode.get("price").asInt();

            System.out.println(key + ":" + valueInt);
        }

        @Test
        public void json노드를_at으로_접근하여_지정된_as형태로_출력() throws JsonProcessingException {
            String carJson = "{ \"brand\" : \"Kia\" , \"price\" : 60}";
            JsonNode jNode = objectMapper.readTree(carJson);

            String key = jNode.at("/brand").asText();
            Integer valueInt = jNode.at("/price").asInt();

            System.out.println(key + ":" + valueInt);
        }

        @Test
        public void json노드를_findPath로_주어진_필드값을_찾아_출력() throws JsonProcessingException {
            String jsonArray = "[{ \"brand\" : \"Kia\" , \"price\" : 60},{ \"brand\" : \"hyundai\" , \"price\" : 70}]";
            JsonNode jNode = objectMapper.readTree(jsonArray);

            JsonNode path = jNode.get(1).findPath("price");

            System.out.println(path.asText());
        }

        @Test
        public void json노드를_findValue로_주어진_필드값을_찾아_출력() throws JsonProcessingException {
            String jsonArray = "[{ \"brand\" : \"Kia\" , \"price\" : 60},{ \"brand\" : \"hyundai\" , \"price\" : 70}]";
            JsonNode jNode = objectMapper.readTree(jsonArray);

            JsonNode value = jNode.get(1).findValue("brand");

            System.out.println(value.asText());
        }

        @Test
        public void json노드를_findValues로_주어진_필드_리스트를_찾아_출력() throws JsonProcessingException {
            String jsonArray = "[{ \"brand\" : \"Kia\" , \"price\" : 60},{ \"brand\" : \"hyundai\" , \"price\" : 70}]";
            JsonNode jNode = objectMapper.readTree(jsonArray);

            List<JsonNode> values = jNode.findValues("brand");

            System.out.println(values);
        }

        @Test
        public void json배열에서_단일json객체_가져오기() throws JsonProcessingException {
            String jsonArray = "[{ \"brand\" : \"Kia\" , \"price\" : 60},{ \"brand\" : \"hyundai\" , \"price\" : 70}]";
            JsonNode jNode = objectMapper.readTree(jsonArray);

            System.out.println(jNode.get(1));
        }

        @Test
        public void getNodeType_으로_json_타입을_검증한다() throws JsonProcessingException {
            String jsonArray = "{ \"brand\" : \"Kia\" , \"price\" : 60}";
            JsonNode jNode = objectMapper.readTree(jsonArray);

            if (jNode.get("brand").isTextual()){
                System.out.println("문자열입니다.");
            }
            if (jNode.get("price").isInt()){
                System.out.println("정수협입니다.");
            }
        }

        /**
         *
         * ArrayNode 클래스 사용법
         * */

        @Test
        public void json배열에_add를_사용하여_요소_추가하기() throws JsonProcessingException {
            String jsonArray = "[{ \"brand\" : \"Kia\" , \"price\" : 60},{ \"brand\" : \"hyundai\" , \"price\" : 70}]";
            ArrayNode arrNode = (ArrayNode)objectMapper.readTree(jsonArray);

            System.out.println(arrNode);

            String addJson = "{ \"brand\" : \"SamSung\" , \"price\" : 100}";
            JsonNode jsonNode = objectMapper.readTree(addJson);

            arrNode.add(jsonNode);

            System.out.println(arrNode);
        }

        @Test
        public void json배열에_insert를_사용하여_지정한_인덱스에_요소_추가하기() throws JsonProcessingException {
            String jsonArray = "[{ \"brand\" : \"Kia\" , \"price\" : 60},{ \"brand\" : \"hyundai\" , \"price\" : 70}]";
            ArrayNode arrNode = (ArrayNode)objectMapper.readTree(jsonArray);

            System.out.println(arrNode);

            String addJson = "{ \"brand\" : \"SamSung\" , \"price\" : 100}";
            JsonNode jsonNode = objectMapper.readTree(addJson);

            arrNode.insert(1,jsonNode);

            System.out.println(arrNode);
        }

        @Test
        public void json배열에_remove를_사용하여_요소_제거하기() throws JsonProcessingException {
            String jsonArray = "[{ \"brand\" : \"Kia\" , \"price\" : 60},{ \"brand\" : \"hyundai\" , \"price\" : 70}]";
            ArrayNode arrNode = (ArrayNode)objectMapper.readTree(jsonArray);

            System.out.println(arrNode);

            arrNode.remove(1);

            System.out.println(arrNode);
        }


        /**
         * 
         * CsvMapper 사용법
         * */

        @Test
        public void json배열을_csv로_변환() throws IOException {
            String jsonArray = "[{ \"brand\" : \"Kia\" , \"price\" : 60}," +
                    "           { \"brand\" : \"hyundai\" , \"price\" : 70}]";
            ArrayNode jNode = (ArrayNode)objectMapper.readTree(jsonArray);

            CsvMapper csvMapper = new CsvMapper();
            CsvSchema csvSchema = csvMapper
                                    .schemaFor(Car.class)
                                    .withColumnSeparator(',')
                                    .withHeader();

            ObjectWriter ow = csvMapper.writer(csvSchema);
            ow.writeValue(System.out, jNode);
            ow.writeValue(new File("src\\data\\target.csv"), jNode);
        }

        @Test
        public void csv파일을_읽어_json타입으로_출력() throws IOException {
            CsvMapper csvMapper = new CsvMapper();
            CsvSchema csvSchema = CsvSchema
                                    .emptySchema()
                                    .withHeader();

            ObjectReader or = csvMapper
                                .readerFor(Car.class)
                                .with(csvSchema);

            InputStream is = new FileInputStream(new File("src\\data\\target.csv"));
            MappingIterator<Car> mi = or.readValues(is);

            List<Car> cars = mi.readAll();

            cars.stream()
                    .forEach(System.out::println);

            is.close();
        }

        /**
         *
         * Jackson 어노테이션 사용법
         * */

        @Test
        public void jsonPropertyOrder_사용하여_이름의_순서대로_직렬화한다() throws IOException {

            @JsonPropertyOrder({"brand","price"})
            @AllArgsConstructor
            @Data
            class InnerCar {
                private String brand;
                private int price;

            }
            InnerCar innerCar = new InnerCar("Genesis", 100);
            objectMapper.writeValue(System.out, innerCar);
        }

        @Test
        public void jsonFormat으로_원하는_날짜형식을_지정한다() throws IOException {
            @JsonPropertyOrder({"brand","price"})
            @AllArgsConstructor
            @Data
            class InnerCar {
                private String brand;
                private int price;

                @JsonFormat(shape = JsonFormat.Shape.STRING, pattern = "yyyy-MM-dd")
                private Date date;
            }
            InnerCar innerCar = new InnerCar("Genesis", 100, new Date());
            objectMapper.writeValue(System.out, innerCar);
        }

        @Test
        public void json에_Root_Node를_부여한다() throws IOException {
            @JsonRootName("Global_car")
            @AllArgsConstructor
            @Data
            class InnerCar {
                private String brand;
                private int price;

                @JsonFormat(shape = JsonFormat.Shape.STRING, pattern = "yyyy-MM-dd")
                private Date date;
            }
            InnerCar innerCar = new InnerCar("Genesis", 100, new Date());
            objectMapper.enable(SerializationFeature.WRAP_ROOT_VALUE)
                    .writeValue(System.out, innerCar);
        }
}
