import React, { useState, useEffect } from 'react';
import { ChevronLeft, ChevronRight, RotateCcw, Brain, MessageCircle, Zap, CheckCircle2 } from 'lucide-react';

// --- DATA SOURCE ---
const QUIZ_DATA = {
  basic_verbs: [
    // Day 1
    { day: 1, source: "대표", kr: "반려동물 키우시나요?", en: "Do you have any pets?" },
    { day: 1, source: "교재1", kr: "내 조카는 거북이를 키운다.", en: "My nephew has a turtle." },
    { day: 1, source: "교재1", kr: "나는 시카고에 사는 친구들이 있다.", en: "I have some friends living in Chicago." },
    { day: 1, source: "교재1", kr: "남동생과 나는 우애가 돈독하다.", en: "My brother and I have a strong bond." },
    { day: 1, source: "교재1", kr: "내 생각은 달랐다.", en: "I had a different idea." },
    { day: 1, source: "교재1", kr: "친구들과 술래잡기하면서 놀던 즐거운 추억이 있다.", en: "I have some fond memories of playing tag with my friends." },
    { day: 1, source: "교재2", kr: "프로젝트가 끝나서 좋죠?", en: "Are you glad the project is over?" },
    { day: 1, source: "교재2", kr: "솔직히 시원섭섭해요. 일을 끝내서 홀가분하긴 한데, 팀이 그리울 것 같아요.", en: "Honestly, I have mixed feelings. I’m relieved we finished the work, but I’ll miss the team." },
    { day: 1, source: "교재2", kr: "음대 대신 의대를 선택한 거 맞지?", en: "You chose med school over music, right?" },
    { day: 1, source: "교재2", kr: "응, 힘든 결정이었어.", en: "Yeah, it was a tough decision." },
    { day: 1, source: "교재2", kr: "음악을 계속하지 않은 걸 후회해?", en: "Do you have any regrets about not pursuing music?" },
    { day: 1, source: "교재2", kr: "조금은 그렇지만, 되도록 생각 안 하려고 해", en: "A few, but I try not to think about it too much." },
    { day: 1, source: "교재3", kr: "그녀는 공포 영화보다는 영어 강의를 택할 겁니다.", en: "She would choose an English lecture over any horror movie." },
    { day: 1, source: "교재3", kr: "어떤 사람들은 가구를 살 때 가격보다 품질을 중시한다.", en: "Some people choose quality over price when buying furniture." },
    { day: 1, source: "교재3", kr: "어떻게 내가 아닌 그 사람을 선택할 수가 있어!", en: "I can’t believe you chose him over me!" },
    { day: 1, source: "교재3", kr: "장기적인 목표보다 눈앞의 즐거움을 선택하는 사람들이 있다.", en: "Some people choose short-term pleasure over long-term goals." },
    // Day 2
    { day: 2, source: "대표", kr: "방금 간식을 먹었더니, 배가 별로 안 고파요.", en: "I just had a snack, so I’m not that hungry." },
    { day: 2, source: "교재1", kr: "나는 아침으로 주로 시리얼을 먹는다.", en: "I usually have cereal for breakfast." },
    { day: 2, source: "교재1", kr: "저는 스테이크 먹을게요.", en: "I’ll have the steak, please." },
    { day: 2, source: "교재1", kr: "오늘 점심으로 아무것도 안 먹었어요.", en: "I didn’t have anything for lunch today." },
    { day: 2, source: "교재1", kr: "출발하기 전에 뭐라도 먹어야 해.", en: "You should have something before you leave." },
    { day: 2, source: "교재1", kr: "피자 먹으면서 영화 보자.", en: "Let’s have some pizza and watch a movie." },
    { day: 2, source: "교재2", kr: "수업 전에 뭐 좀 먹었어?", en: "Did you have anything before class?" },
    { day: 2, source: "교재2", kr: "그냥 바나나 하나 먹었어. 시간이 없었어.", en: "Just a banana. I was in a rush." },
    { day: 2, source: "교재2", kr: "점심은 주로 몇 시에 먹어요?", en: "What time do you usually have lunch?" },
    { day: 2, source: "교재2", kr: "그렇게 안 바쁘면 대략 12시쯤에요.", en: "Around noon, if I’m not too busy." },
    { day: 2, source: "교재2", kr: "오, 저도 그런데! 오늘 저랑 뭐 간단히 먹을래요?", en: "Oh, me too! Want to grab something together today?" },
    { day: 2, source: "교재2", kr: "좋지요! 한국 음식이라면 좋아요.", en: "Sure! I could go for some Korean food." },
    { day: 2, source: "교재3", kr: "어제 공원에서 널 봤어.", en: "I saw you at the park yesterday." },
    { day: 2, source: "교재3", kr: "어제 공원에서 널 지켜봤어.", en: "I watched you at the park yesterday." },
    { day: 2, source: "교재3", kr: "술집에 있는 TV에서 경기가 나오던데, 집중해서 보지는 않았어.", en: "I saw the game on the TV at the bar, but I didn’t watch it." },
    { day: 2, source: "교재3", kr: "그 영화를 대형 화면으로 봤어.", en: "I saw the movie on the big screen." },
    { day: 2, source: "교재3", kr: "어젯밤에 넷플릭스 다큐멘터리를 보고 있는데 엄마한테 전화가 왔다.", en: "My mom called while I was watching a Netflix documentary last night." },
    // Day 3
    { day: 3, source: "대표", kr: "어제 아침에는 별일이 다 있었어요.", en: "I had a weird morning yesterday." },
    { day: 3, source: "교재1", kr: "우리는 지난주 일본에서 정말 좋은 시간을 보냈어요.", en: "We had a great time in Japan last week." },
    { day: 3, source: "교재1", kr: "나는 힘든 유년 시절을 보냈다.", en: "I had a rough childhood." },
    { day: 3, source: "교재1", kr: "(줌 회의에서) 제가 접속 상태가 안 좋은 것 같습니다.", en: "I think I have a bad connection." },
    { day: 3, source: "교재1", kr: "(친한 친구를 언급하며) 저희는 만나면 늘 대화가 즐겁습니다.", en: "We always have such good conversations." },
    { day: 3, source: "교재1", kr: "그는 비행기를 오래 타서, 잠을 좀 더 자야 해요.", en: "He had a long flight, so he needs some extra sleep." },
    { day: 3, source: "교재2", kr: "여전히 예술 학교에 진학할 계획이니?", en: "Are you still planning to go to art school?" },
    { day: 3, source: "교재2", kr: "잘 모르겠어. 전공 때문에 엄마랑 사이가 틀어졌거든.", en: "I’m not sure. My mom and I had a falling-out over my major." },
    { day: 3, source: "교재2", kr: "안녕, 최근에 잘 안 보이더라.", en: "Hey, I haven’t seen you around lately." },
    { day: 3, source: "교재2", kr: "응∙∙∙ 한 주 동안 마음이 좀 힘들었거든.", en: "I know... I had a rough week, emotionally." },
    { day: 3, source: "교재2", kr: "무슨 일인지 이야기해 줄래?", en: "Want to talk about it?" },
    { day: 3, source: "교재2", kr: "음, 다음에. 우선 머리 좀 식힐 시간이 필요해서.", en: "Maybe later. I just need some time to clear my head." },
    { day: 3, source: "교재3", kr: "내가 어렸을 때, 우리 이웃이 나무에서 떨어졌어요.", en: "My neighbor fell out of a tree when I was young." },
    { day: 3, source: "교재3", kr: "아무래도 버스에서 내리다 핸드폰이 주머니에서 빠진 것 같아.", en: "I think my phone fell out of my pocket while I was getting off the bus." },
    { day: 3, source: "교재3", kr: "그거 못 들었어? 그 두 사람 크리스마스 이후로 사이가 완전히 틀어졌대.", en: "Didn’t you hear? Those two had a major falling-out after Christmas." },
    { day: 3, source: "교재3", kr: "제 동업자와 저는 돈 문제로 사이가 멀어졌어요.", en: "My business partner and I had a falling-out over money." },
    // Day 4
    { day: 4, source: "대표", kr: "삼성이 오늘 신제품 출시 행사를 열어요.", en: "Samsung is having a new-product launch today." },
    { day: 4, source: "교재1", kr: "제가 이번 주말에 집들이를 합니다.", en: "I’m having a housewarming party this weekend." },
    { day: 4, source: "교재1", kr: "백화점들이 이번 주에 일제히 세일을 한다.", en: "The department stores are all having sales this week." },
    { day: 4, source: "교재1", kr: "우리는 6월에 코엑스에서 채용 박람회를 열었다.", en: "We had a job fair at COEX in June." },
    { day: 4, source: "교재1", kr: "이번 주말에 모임이 있으니 오고 싶으면 오렴.", en: "We’re having a get-together this weekend if you want to join." },
    { day: 4, source: "교재1", kr: "그 제과점에서 오늘 원 플러스 원 행사를 한다.", en: "The bakery has a buy-one-get-one deal today." },
    { day: 4, source: "교재2", kr: "여행용 가방을 새로 사야 해.", en: "I need to buy some new luggage." },
    { day: 4, source: "교재2", kr: "아, 코스트코에서 지금 샘소나이트 여행 가방 세일 중인데.", en: "Oh, Costco is having a sale on Samsonite luggage right now." },
    { day: 4, source: "교재2", kr: "준호 씨, 브라질 생활 어때요?", en: "How is life in Brazil treating you, Junho?" },
    { day: 4, source: "교재2", kr: "쉽지는 않지만 나아지고 있습니다.", en: "It’s not easy, but it’s getting better." },
    { day: 4, source: "교재2", kr: "지금쯤이면 친구도 사귀었겠어요?", en: "Do you have any friends yet?" },
    { day: 4, source: "교재2", kr: "네, 아파트 단지에 한국인 가정이 몇 있는데 매주 금요일마다 바비큐 파티를 해요.", en: "Yeah, a couple of Korean families live in my apartment building and we have barbeques every Friday." },
    { day: 4, source: "교재3", kr: "퇴근은 한 거지?", en: "Did you leave work yet?" },
    { day: 4, source: "교재3", kr: "엄마한테 전화한 거지?", en: "Did you call your mom yet?" },
    { day: 4, source: "교재3", kr: "저녁은 먹은 거지?", en: "Did you eat dinner yet?" },
    { day: 4, source: "교재3", kr: "집안일은 다 한 거지?", en: "Did you finish your chores yet?" },
    { day: 4, source: "교재3", kr: "당신 이미 집 나선 거지?", en: "Did you leave yet?" },
    { day: 4, source: "교재3", kr: "지갑을 두고 왔네. 아래쪽으로 던져 줄 수 있을까?", en: "I forgot my wallet. Could you throw it down to me, please?" },
    // Day 5
    { day: 5, source: "대표", kr: "오후에는 집안일을 좀 해야 해요.", en: "I have some chores to do in the afternoon." },
    { day: 5, source: "교재1", kr: "할 일이 산더미같이 쌓여 있다.", en: "I have tons of work to do." },
    { day: 5, source: "교재1", kr: "답장해야 할 이메일이 몇 개 있다.", en: "I have a few emails to reply to." },
    { day: 5, source: "교재1", kr: "오늘 오후에 몇 가지 볼일이 좀 있다.", en: "I have some errands to run this afternoon." },
    { day: 5, source: "교재1", kr: "금요일까지 프로젝트 하나를 마무리해야 한다.", en: "We have a project to finish by Friday." },
    { day: 5, source: "교재1", kr: "크리스마스 (선물) 쇼핑을 좀 해야 한다.", en: "I have some Christmas shopping to do." },
    { day: 5, source: "교재2", kr: "안녕하세요, 제리. 우리 방금 프로젝트 진행 상황에 대해 이야기하고 있었어요.", en: "Good morning, Jerry. We were just talking about the project updates." },
    { day: 5, source: "교재2", kr: "네, 늦어서 죄송해요. 뭐 좀 처리할 게 있었어요.", en: "Yes, sorry I’m late. I had something to take care of." },
    { day: 5, source: "교재2", kr: "오늘 바빠? 점심 먹자.", en: "Are you busy today? Let’s have lunch." },
    { day: 5, source: "교재2", kr: "안 돼. 우체국 가는 길이야.", en: "I can’t. I’m on my way to the post office." },
    { day: 5, source: "교재2", kr: "중고나라에서 바지 다섯 개를 팔아서, 택배 부칠 게 많아.", en: "I just sold five pairs of pants on Joonggonara, so I have a lot of packages to ship." },
    { day: 5, source: "교재3", kr: "이번 주말에 친구 고양이를 돌봐 주기로 했어.", en: "I’m taking care of my friend’s cat this weekend." },
    { day: 5, source: "교재3", kr: "저희 아빠는 건강을 좀 더 신경 쓰셔야 해요.", en: "My dad needs to take better care of his health." },
    { day: 5, source: "교재3", kr: "휴가 가기 전에 처리해야 할 일이 좀 있어.", en: "I have some work to take care of before I start my vacation." },
    { day: 5, source: "교재3", kr: "저희 엄마는 주로 오후 시간에 정원을 손질하십니다.", en: "My mom usually takes care of her garden in the afternoons." },
    { day: 5, source: "교재3", kr: "회사에서 모든 경비를 다 대 줬어요.", en: "The company took care of everything." },
  ],
  conversation: [
    // Day 1
    { day: 1, source: "교재1", kr: "저는 재택근무 체질이 아니에요. 늘 딴짓하게 되거든요", en: "Working from home isn’t for me. I always get distracted." },
    { day: 1, source: "교재1", kr: "소개팅은 저랑 안 맞아요.", en: "Going on blind dates isn’t for me." },
    { day: 1, source: "교재1", kr: "노트북은 저랑 좀 안 맞아요. 키보드가 뭔가 엄청 불편하거든요.", en: "Laptops aren’t really for me. Something about the keyboards is super uncomfortable." },
    { day: 1, source: "교재1", kr: "전기차는 좀 별로예요. 충전소는 요즘 늘었지만, 여전히 엄청 귀찮게 느껴져요.", en: "Electric cars aren’t for me. We have more charging stations around now, but it still feels like too much of a hassle." },
    { day: 1, source: "교재1", kr: "그 사람 직업이 좋은 건 아는데, 그런 남자는 나는 별로야.", en: "I know he has a decent job, but guys like him aren’t really for me." },
    { day: 1, source: "교재2", kr: "우리 나가서 맛난 회 먹을까? 내가 살 게.", en: "Why don’t we go out and get some nice sashimi? My treat!" },
    { day: 1, source: "교재2", kr: "너무 고맙긴 한데. 난 회를 별로 안 좋아해. 식감이 적응이 안 돼.", en: "It’s kind of you to offer, but raw fish just isn’t for me. I can’t get used to the texture." },
    { day: 1, source: "교재2", kr: "청취 연습을 위해 <기묘한 이야기>를 시청할 것을 추천합니다.", en: "I recommend watching Stranger Things to practice listening." },
    { day: 1, source: "교재2", kr: "좋은 생각이긴 한데, 저는 미국 프로그램이 체질에 안 맞아요. 스토리에 재미가 안 붙어요.", en: "It’s a good idea, but American shows aren’t for me. I can’t really get into the stories." },
    { day: 1, source: "교재2", kr: "애들하고 정말 잘 노는군요. 선생님 할 생각은 해 보셨나요?", en: "You’re really great around kids. Have you ever thought of being a teacher?" },
    { day: 1, source: "교재2", kr: "아니요. 저는 가르치는 거랑 잘 안 맞아요.", en: "No, no. Teaching isn’t really for me." },
    { day: 1, source: "교재3", kr: "생일 선물로 받은 로잉 머신... 나랑은 별로 안 맞더라고", en: "Do you remember that rowing machine I got for my birthday? Turns out it’s not really for me." },
    { day: 1, source: "교재4", kr: "학과장을 안 하는 게 너랑 맞는 거야", en: "Not being chair suits you." },
    { day: 1, source: "교재4", kr: "혼자 일하는 건 나랑 안 맞는다는 걸 느꼈어.", en: "I’ve found that working on my own doesn’t really suit me." },
    { day: 1, source: "대표", kr: "재택근무는 저랑 안 맞아요.", en: "Working from home isn’t for me." },
    // Day 2
    { day: 2, source: "교재1", kr: "다음 에피소드는 어떤 내용일지 궁금해 미치겠어.", en: "I can’t wait to see what the next episode will bring." },
    { day: 2, source: "교재1", kr: "이 프로젝트가 빨리 끝났으면 좋겠어요. 너무 오래 걸립니다.", en: "I can’t wait to be done with this project. It’s taking forever." },
    { day: 2, source: "교재2", kr: "그 책 드디어 영화로 만들었다며? 응! 어서 보고 싶어.", en: "Yes! I can’t wait to see it." },
    { day: 2, source: "교재4", kr: "말씀 많이 들었습니다. 하루빨리 함께 일하고 싶습니다.", en: "I’ve heard a great deal about you. I look forward to working with you." },
    { day: 2, source: "대표", kr: "하루빨리 새 집으로 이사 가고 싶어요.", en: "I can’t wait to move into the new house." },
    // Day 3
    { day: 3, source: "교재1", kr: "제가 마지막 남은 피자 한 조각 먹어도 될까요?", en: "Do you mind if I finish off the last piece of pizza?" },
    { day: 3, source: "교재1", kr: "에어컨 좀 약하게 하면 안 될까요? 좀 추워서요.", en: "Do you mind turning down the air-conditioning? I feel a bit cold." },
    { day: 3, source: "교재2", kr: "죄송한데, 회의를 금요일로 옮겨도 될까요?", en: "Do you mind if we move the meeting to Friday?" },
    { day: 3, source: "대표", kr: "죄송한데 조금 짧게 해 주시겠어요?", en: "Do you mind keeping it a bit short?" },
    // Day 4
    { day: 4, source: "교재1", kr: "서울은 어디라도 다 너무 비싸. 근데 후암동은 상대적으로 저렴한 편이지", en: "All the neighborhoods in Seoul are super expensive, but Huam-dong is relatively cheap" },
    { day: 4, source: "교재2", kr: "그러게. 요새 물가가 너무너무 비싸.", en: "Yeah. Everything is getting super expensive." },
    { day: 4, source: "대표", kr: "물가가 올라도 너무 올라요.", en: "Everything is getting super expensive." },
    // Day 5
    { day: 5, source: "교재1", kr: "성형 수술 하는 거 어떻게 생각하세요?", en: "How do you feel about plastic surgery?" },
    { day: 5, source: "교재4", kr: "완전 깜박했어.", en: "It totally slipped my mind." },
    { day: 5, source: "대표", kr: "중고차 같은 중고 물품 사는 거 어떻게 생각하세요?", en: "How do you feel about buying used items?" },
  ],
  phrasal_verbs: [
    // Day 1
    { day: 1, source: "교재1", kr: "뭔가 앞뒤가 안 맞잖아.", en: "Something doesn’t add up." },
    { day: 1, source: "교재1", kr: "자, 이 숫자들을 더해 보자. 5 더하기 3은 뭘까?", en: "Let’s add up these numbers now. What’s five plus three?" },
    { day: 1, source: "교재1", kr: "월 10만 원도 쌓이면 4년 후에 거의 5백만 원이 된다.", en: "Just 100,000 won a month will add up to almost 5 million won in four years." },
    { day: 1, source: "교재2", kr: "한 달에 백 달러도 5년이면 6천 달러야.", en: "A hundred bucks a month will add up to $6,000 in five years." },
    { day: 1, source: "교재3", kr: "저를 믿어 보세요. 이게 쌓이면 정말 큽니다.", en: "Trust me, it really adds up." },
    // Day 2
    { day: 2, source: "교재1", kr: "바람이 워낙 강해서 눈이 다 날아가 버렸더군.", en: "The wind was so strong that it blew the snow away." },
    { day: 2, source: "교재1", kr: "이번에 새로 나온 태블릿을 처음 봤을 때 매우 인상적이었어.", en: "When I first saw their new tablet, I was blown away." },
    { day: 2, source: "교재3", kr: "메뉴 볼 때마다 깜짝 놀라. 만 오천 원 이하가 없다니까.", en: "I’m blown away every time I look at a menu." },
    // Day 3
    { day: 3, source: "교재1", kr: "몇 년을 매일 썼더니 컴퓨터가 결국 고장이 났다.", en: "The computer finally broke down after using it daily for years." },
    { day: 3, source: "교재1", kr: "담배꽁초가 분해되는 데 18개월에서 10년이 걸리는 거 알았어?", en: "Did you know that cigarette butts take between 18 months and 10 years to break down?" },
    { day: 3, source: "교재2", kr: "정신적으로 완전히 무너져서 아무 말도 안 나왔어.", en: "I just broke down completely and couldn’t even get words out." },
    // Day 4
    { day: 4, source: "교재1", kr: "여기 신호가 끊겨. 나이트클럽이거든.", en: "My signal is breaking up down here. I’m in a nightclub." },
    { day: 4, source: "교재2", kr: "좀 더 짧은 클립으로 쪼개야 해.", en: "You need to break it up into smaller clips." },
    { day: 4, source: "교재3", kr: "최근에 헤어지게 되었다.", en: "They recently broke up." },
    // Day 5
    { day: 5, source: "교재1", kr: "다음 달에 파리로 여행 가는데, 프랑스어 복습 좀 해야겠어.", en: "I am travelling to Paris next month, so I think I need to brush up on my French." },
    { day: 5, source: "교재2", kr: "가기 전에 한국 역사 복습 좀 해야겠어.", en: "I want to brush up on my Korean history before we go." },
    { day: 5, source: "교재3", kr: "작업 멘트를 연습해 보는 것이 내가 생각할 수 있는 전부였다.", en: "All I could think to do was brush up on some pickup lines." },
  ]
};

const SOURCE_LABELS = {
  "대표": "대표예제",
  "교재1": "Model examples",
  "교재2": "Small Talk",
  "교재3": "Cases in point",
  "교재4": "Further studies"
};

const CURRICULUM_LABELS = {
  basic_verbs: "기본동사100",
  conversation: "영어회화100",
  phrasal_verbs: "구동사100"
};

export default function App() {
  const [tab, setTab] = useState('basic_verbs');
  const [view, setView] = useState('main'); // main, selection, quiz, result
  const [difficulty, setDifficulty] = useState(null); // mild, spicy
  const [currentCards, setCurrentCards] = useState([]);
  const [currentIndex, setCurrentIndex] = useState(0);
  const [isFlipped, setIsFlipped] = useState(false);

  // Fisher-Yates shuffle
  const shuffle = (array) => {
    const a = [...array];
    for (let i = a.length - 1; i > 0; i--) {
      const j = Math.floor(Math.random() * (i + 1));
      [a[i], a[j]] = [a[j], a[i]];
    }
    return a;
  };

  // Filter and Pick Logic
  const startQuiz = (diff) => {
    let pool = QUIZ_DATA[tab] ?? [];

    // Filter by difficulty (Mild: 대표, 교재1 only)
    if (diff === 'mild') {
      pool = pool.filter(item => item.source === '대표' || item.source === '교재1');
    }

    const shuffled = shuffle(pool);
    const selected = shuffled.slice(0, Math.min(10, shuffled.length));

    setCurrentCards(selected);
    setCurrentIndex(0);
    setIsFlipped(false);
    setDifficulty(diff);
    // show selection --> quiz; view may render before state is applied, so quiz view guards against empty cards
    setView('quiz');
  };

  const handleNext = () => {
    if (currentIndex < currentCards.length - 1) {
      setIsFlipped(false);
      // small timeout for flip animation feel
      setTimeout(() => setCurrentIndex((i) => i + 1), 200);
    } else {
      setView('result');
    }
  };

  const handlePrev = () => {
    if (currentIndex > 0) {
      setIsFlipped(false);
      setTimeout(() => setCurrentIndex((i) => i - 1), 200);
    }
  };

  const resetAll = () => {
    setView('main');
    setDifficulty(null);
    setCurrentCards([]);
    setCurrentIndex(0);
    setIsFlipped(false);
  };

  // Keyboard support when in quiz
  useEffect(() => {
    if (view !== 'quiz') return;

    const onKey = (e) => {
      if (e.key === 'ArrowRight') {
        e.preventDefault();
        handleNext();
      } else if (e.key === 'ArrowLeft') {
        e.preventDefault();
        handlePrev();
      } else if (e.key === ' ' || e.key === 'Spacebar' || e.key === 'Enter') {
        // flip card
        // prevent page scroll on space
        e.preventDefault();
        setIsFlipped((f) => !f);
      }
    };

    window.addEventListener('keydown', onKey);
    return () => window.removeEventListener('keydown', onKey);
  }, [view, currentIndex, currentCards.length]);

  // Rendering Components
  const TabButton = ({ id, label, icon: Icon }) => (
    <button
      onClick={() => { setTab(id); setView('selection'); }}
      className={`flex flex-col items-center justify-center p-6 rounded-2xl transition-all duration-300 ${
        tab === id && view !== 'main' 
          ? 'bg-blue-600 text-white shadow-xl scale-105' 
          : 'bg-white text-slate-600 hover:bg-blue-50 border border-slate-100'
      }`}
      aria-pressed={tab === id}
    >
      <Icon size={32} className="mb-3" />
      <span className="font-bold text-lg">{label}</span>
    </button>
  );

  return (
    <div className="min-h-screen bg-slate-50 flex flex-col items-center p-4 md:p-8 font-sans text-slate-900">
      {/* Header */}
      <header className="w-full max-w-4xl mb-8 flex justify-between items-center">
        <h1 className="text-2xl font-black text-blue-700 tracking-tighter flex items-center gap-2">
          <Zap fill="currentColor" /> SPEED QUIZ
        </h1>
        {view !== 'main' && (
          <button 
            onClick={resetAll}
            className="text-slate-500 hover:text-blue-600 flex items-center gap-1 font-semibold transition-colors"
          >
            <RotateCcw size={18} /> 처음으로
          </button>
        )}
      </header>

      {/* Main Content */}
      <main className="w-full max-w-2xl">
        
        {/* VIEW: MAIN (Curriculum Selection) */}
        {view === 'main' && (
          <div className="grid grid-cols-1 md:grid-cols-3 gap-4 animate-in fade-in slide-in-from-bottom-4">
            <TabButton id="basic_verbs" label="기본동사100" icon={Brain} />
            <TabButton id="conversation" label="영어회화100" icon={MessageCircle} />
            <TabButton id="phrasal_verbs" label="구동사100" icon={Zap} />
          </div>
        )}

        {/* VIEW: SELECTION (Day / Difficulty) */}
        {view === 'selection' && (
          <div className="bg-white p-8 rounded-3xl shadow-sm border border-slate-100 animate-in zoom-in-95 duration-300">
            <div className="text-center mb-8">
              <span className="bg-blue-100 text-blue-700 px-4 py-1 rounded-full text-sm font-bold uppercase tracking-wider mb-2 inline-block">
                {CURRICULUM_LABELS[tab]}
              </span>
              <h2 className="text-3xl font-extrabold text-slate-800">Week 1 (Day 1-5)</h2>
              <p className="text-slate-500 mt-2">난이도를 선택해 주세요</p>
            </div>
            
            <div className="grid grid-cols-1 gap-4">
              <button
                onClick={() => startQuiz('mild')}
                className="group relative overflow-hidden bg-green-50 hover:bg-green-100 border-2 border-green-200 p-6 rounded-2xl transition-all"
              >
                <div className="flex justify-between items-center">
                  <div className="text-left">
                    <h3 className="text-xl font-bold text-green-800">순한맛 (Mild)</h3>
                    <p className="text-green-600 text-sm">대표예제 & Model examples 위주</p>
                  </div>
                  <div className="bg-green-500 text-white p-2 rounded-full group-hover:scale-110 transition-transform">
                    <CheckCircle2 size={24} />
                  </div>
                </div>
              </button>

              <button
                onClick={() => startQuiz('spicy')}
                className="group relative overflow-hidden bg-red-50 hover:bg-red-100 border-2 border-red-200 p-6 rounded-2xl transition-all"
              >
                <div className="flex justify-between items-center">
                  <div className="text-left">
                    <h3 className="text-xl font-bold text-red-800">매운맛 (Spicy)</h3>
                    <p className="text-red-600 text-sm">전체 출처에서 10문장 랜덤 추출</p>
                  </div>
                  <div className="bg-red-500 text-white p-2 rounded-full group-hover:scale-110 transition-transform">
                    <Zap size={24} />
                  </div>
                </div>
              </button>
            </div>
          </div>
        )}

        {/* VIEW: QUIZ (Flashcards) */}
        {view === 'quiz' && (
          <div className="flex flex-col items-center">
            {/* If cards are not yet prepared, show a small placeholder */}
            {currentCards.length === 0 ? (
              <div className="bg-white p-8 rounded-xl shadow-sm w-full text-center">
                준비 중... 잠시만 기다려 주세요.
              </div>
            ) : (
              <>
                {/* Progress Bar */}
                <div className="w-full h-2 bg-slate-200 rounded-full mb-6 overflow-hidden">
                  <div 
                    className={`h-full transition-all duration-500 ${difficulty === 'mild' ? 'bg-green-500' : 'bg-red-500'}`}
                    style={{ width: `${((currentIndex + 1) / (currentCards.length || 1)) * 100}%` }}
                  />
                </div>

                <div className="flex justify-between w-full mb-4 px-2">
                  <span className="font-bold text-slate-400">Question {currentIndex + 1} / {currentCards.length}</span>
                  <span className={`font-bold ${difficulty === 'mild' ? 'text-green-600' : 'text-red-600'}`}>
                    {difficulty === 'mild' ? 'Mild Mode' : 'Spicy Mode'}
                  </span>
                </div>

                {/* Flashcard Container */}
                <div 
                  className="relative w-full aspect-[4/3] cursor-pointer perspective-1000 group"
                  onClick={() => setIsFlipped((f) => !f)}
                >
                  <div
                    // container that gets keyboard focus and is announced as a toggle button
                    role="button"
                    tabIndex={0}
                    aria-pressed={isFlipped}
                    onKeyDown={(e) => {
                      if (e.key === 'Enter' || e.key === ' ') {
                        e.preventDefault();
                        setIsFlipped((f) => !f);
                      }
                    }}
                    className={`relative w-full h-full transition-all duration-500 preserve-3d ${isFlipped ? 'rotate-y-180' : ''}`}
                  >
                    
                    {/* FRONT: Korean */}
                    <div className="absolute inset-0 w-full h-full backface-hidden bg-white rounded-3xl shadow-xl border-b-8 border-slate-200 flex flex-col items-center justify-center p-8 text-center">
                      <span className="text-slate-300 text-sm font-bold absolute top-6 tracking-widest uppercase">Meaning</span>
                      <p className="text-2xl md:text-3xl font-bold text-slate-800 leading-tight">
                        {currentCards[currentIndex]?.kr}
                      </p>
                      <p className="text-slate-400 mt-6 text-sm animate-pulse">카드를 터치하여 영어 확인</p>
                    </div>

                    {/* BACK: English + Details */}
                    <div className="absolute inset-0 w-full h-full backface-hidden bg-blue-600 rounded-3xl shadow-xl border-b-8 border-blue-800 flex flex-col items-center justify-center p-8 text-center rotate-y-180">
                      <div className="absolute top-6 left-0 right-0 flex justify-center gap-2">
                        <span className="bg-blue-500/50 text-white px-3 py-1 rounded-full text-xs font-bold">
                          Day {currentCards[currentIndex]?.day}
                        </span>
                        <span className="bg-yellow-400 text-blue-900 px-3 py-1 rounded-full text-xs font-black shadow-sm">
                          {SOURCE_LABELS[currentCards[currentIndex]?.source] ?? currentCards[currentIndex]?.source}
                        </span>
                      </div>
                      
                      <p className="text-2xl md:text-3xl font-black text-white leading-snug">
                        {currentCards[currentIndex]?.en}
                      </p>

                      <div className="absolute bottom-6 text-blue-200 text-xs font-medium">
                        {CURRICULUM_LABELS[tab]}
                      </div>
                    </div>

                  </div>
                </div>

                {/* Controls */}
                <div className="flex gap-4 mt-8 w-full">
                  <button 
                    onClick={(e) => { e.stopPropagation(); handlePrev(); }}
                    disabled={currentIndex === 0}
                    className="flex-1 flex items-center justify-center gap-2 bg-white text-slate-600 py-4 rounded-2xl font-bold shadow-sm disabled:opacity-30 border border-slate-200 hover:bg-slate-50 transition-all"
                    aria-disabled={currentIndex === 0}
                  >
                    <ChevronLeft /> Prev
                  </button>
                  <button 
                    onClick={(e) => { e.stopPropagation(); handleNext(); }}
                    className={`flex-1 flex items-center justify-center gap-2 text-white py-4 rounded-2xl font-bold shadow-lg transition-all scale-105 ${difficulty === 'mild' ? 'bg-green-600 hover:bg-green-700' : 'bg-red-600 hover:bg-red-700'}`}
                  >
                    {currentIndex === currentCards.length - 1 ? 'Finish' : 'Next'} <ChevronRight />
                  </button>
                </div>
              </>
            )}
          </div>
        )}

        {/* VIEW: RESULT */}
        {view === 'result' && (
          <div className="bg-white p-10 rounded-3xl shadow-xl text-center animate-in zoom-in-95 duration-500">
            <div className="w-20 h-20 bg-blue-100 text-blue-600 rounded-full flex items-center justify-center mx-auto mb-6">
              <CheckCircle2 size={48} />
            </div>
            <h2 className="text-3xl font-black text-slate-800 mb-2">학습 완료!</h2>
            <p className="text-slate-500 mb-8">오늘의 10문장 학습을 모두 마쳤습니다.<br/>매일 꾸준히가 가장 중요해요.</p>
            
            <div className="flex flex-col gap-3">
              <button 
                onClick={() => startQuiz(difficulty || 'spicy')}
                className="w-full bg-blue-600 text-white py-4 rounded-2xl font-bold shadow-lg hover:bg-blue-700 transition-all"
              >
                다시 한 번 도전
              </button>
              <button 
                onClick={resetAll}
                className="w-full bg-slate-100 text-slate-600 py-4 rounded-2xl font-bold hover:bg-slate-200 transition-all"
              >
                다른 커리큘럼 선택
              </button>
            </div>
          </div>
        )}

      </main>

      <footer className="mt-auto py-8 text-slate-400 text-sm font-medium">
        &copy; 2024 Speed Quiz English Training
      </footer>

      {/* Global CSS for Flip Animation */}
      <style dangerouslySetInnerHTML={{ __html: `
        .perspective-1000 { perspective: 1000px; }
        .preserve-3d { transform-style: preserve-3d; }
        .backface-hidden { backface-visibility: hidden; }
        .rotate-y-180 { transform: rotateY(180deg); }
      `}} />
    </div>
  );
}
