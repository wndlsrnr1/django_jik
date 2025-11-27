DRF content negotiation과 관련 코드 확인 중.


[3 tools called]


코드 확인 결과, DRF의 Content Negotiation과 쿼리 파라미터 충돌입니다. 관련 코드 확인 중:


[2 tools called]


## 문제 상황 설명

### DRF Content Negotiation

DRF는 `?format=xxx` 쿼리 파라미터를 응답 형식(JSON, HTML 등) 지정에 사용합니다.

예:
- `?format=json` → JSON renderer
- `?format=xml` → XML renderer  
- `?format=html` → HTML renderer

### 현재 코드의 문제

```43:43:projects/serializers/admin_project_serializer.py
    format = serializers.ChoiceField(choices=["csv", "excel"], default="csv")
```

```596:596:projects/views/admin_project_view.py
        export_format = serializer.validated_data["format"]
```

`?format=excel`을 보내면:
1. DRF가 `format=excel`을 renderer 형식으로 해석
2. "excel" renderer가 없어 404 반환
3. 애플리케이션 로직까지 도달하지 못함

### 해결 방법

`format` 파라미터 이름을 `export_format`으로 변경:

1. Serializer 필드명 변경: `format` → `export_format`
2. View에서 참조 변경: `serializer.validated_data["format"]` → `serializer.validated_data["export_format"]`
3. Service 레이어는 이미 `export_format`을 사용 중이므로 변경 없음

이렇게 하면 DRF의 content negotiation과 충돌하지 않고, 내보내기 형식만 지정할 수 있습니다.

**요약**: `format`은 DRF 예약 파라미터이므로, 비즈니스 파라미터는 `export_format` 같은 이름으로 변경해야 합니다.
