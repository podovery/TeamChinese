# TeamChinese
## 퇴직후 치킨집을 차렸지


## hi
chicken
pizza

hamberger

# 코드
	using System.Collections;
	using System.Collections.Generic;
	using UnityEngine;

public class PlayerView : MonoBehaviour
{
    public float viewRadius; //시야 영역 반지름
    [Range(0, 360)]
    public float viewAngle; //시야 영역 각도
    public float meshResolution; //샘플링 비율


    public LayerMask targetMask, obstacleMask; //타겟 마스크, 장애물 마스크

    public List<Transform> visibleTarget = new List<Transform>(); //타겟 마스크에 ray hit된 tranform을 보관 하는 리스트

    void Start()
    {
        StartCoroutine(FindTargetDelay(0.1f)); //코루틴 실행
    }

    IEnumerator FindTargetDelay(float delay)
    {
        while (true)
        {
            yield return new WaitForSeconds(delay); //delay초 기다리기
            FindVisibleTarget();
        }
    }

    void FindVisibleTarget()
    {
        visibleTarget.Clear(); //리스트 초기화
        Collider[] targetsViewRadius = Physics.OverlapSphere(transform.position, viewRadius, targetMask);

        for (int i = 0; i < targetsViewRadius.Length; ++i)
        {
            Transform target = targetsViewRadius[i].transform;
            Vector3 dirToTarget = (target.position - transform.position).normalized;

            if (Vector3.Angle(transform.forward, dirToTarget) < viewAngle / 2) //시아 범위 안에있는가
            {
                float dstToTarget = Vector3.Distance(transform.position, target.transform.position);

                if (!Physics.Raycast(transform.position, dirToTarget, dstToTarget, obstacleMask)) //벽에 가려졌는가
                {
                    visibleTarget.Add(target);
                    targetsViewRadius[i].GetComponent<MeshRenderer>().enabled = true;
                }
                else
                {
                    targetsViewRadius[i].GetComponent<MeshRenderer>().enabled = false;
                }
            }
            else
            {
                targetsViewRadius[i].GetComponent<MeshRenderer>().enabled = false;
            }
        }
    }

    public Vector3 DirFromAngle(float angleDegrees, bool anglelsGlobal)
    {
        if (!anglelsGlobal)
        {
            angleDegrees += transform.eulerAngles.y;
        }
        return new Vector3(Mathf.Cos((-angleDegrees + 90) * Mathf.Deg2Rad), 0, Mathf.Sin((-angleDegrees + 90) * Mathf.Deg2Rad));
    }

	public struct ViewCastInfo
	{
		public bool hit;
		public Vector3 point;
		public float dst;
		public float angle;

		public ViewCastInfo(bool _hit, Vector3 _point, float _dst, float _angle)
		{
			hit = _hit;
			point = _point;
			dst = _dst;
			angle = _angle;
		}
	}

	void DrawPlayerView()
    {
        int stepCount = Mathf.RoundToInt(viewAngle * meshResolution);
        float stepAngleSize = viewAngle / stepCount;

        for (int i = 0; i < stepCount; ++i)
        {
            float angle = transform.eulerAngles.y - viewAngle / 2 + stepAngleSize * i;
            Debug.DrawLine(transform.position, transform.position + DirFromAngle(angle, true) * viewRadius, Color.green);
        }
    }

    void LateUpdate()
    {
        DrawPlayerView();
    }

    ViewCastInfo ViewCast(float globalAngle)
    {
        Vector3 dir = DirFromAngle(globalAngle, true);
        RaycastHit hit;

        if (Physics.Raycast(transform.position, dir, out hit, viewRadius, obstacleMask))
        {
            return new ViewCastInfo(true, hit.point, hit.distance, globalAngle);
        }
        else
        {
            return new ViewCastInfo(false, transform.position + dir * viewRadius, viewRadius, globalAngle);
        }
    }
}
